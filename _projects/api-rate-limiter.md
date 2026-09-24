---
title: API rate limiter
summary: Drop-in quotas in front of an existing API — reverse proxy or middleware — with Redis limits that stay correct under load.
stack:
  - Go
  - Redis
  - gRPC
repo: https://github.com/dgallantino/api-rate-limiter
image: /assets/img/projects/api-rate-limiter/api-ratelimit.png
featured: true
status: shipped
date: 2026-09-19
---

A Check service sits in front of an origin that already works. One call decides if the request is allowed. A deny is HTTP 429, and origin work does not run.

Pattern A is `cmd/proxy`: it calls Check, then forwards the request if the key still has quota. Pattern B is `pkg/httplimit` middleware on the origin, so the handler never runs on a deny. Both dial the same gRPC `Checker`. Policy is a YAML file. Counters are Redis.

`docker compose up` runs the proxy on `:8080`, a limited origin on `:8000`, Check, Redis, Prometheus, and a dashboard on `:8081`. Multi-region, billing, and auth on the dashboard stay out.

## How it is built

At work I kept shipping small, crude, one-off rate limiters. They were usually Python: easy to drop in anywhere, good enough for that deploy. Each one owned its own Redis script and its own idea of what to do when Redis died. I wanted one Check those apps could call, and I wanted the limit to stay correct when many requests arrive together.

### A burst is more goroutines

I wrote Check in Go so a burst is more goroutines on one process, not a worker pool I have to size. `cmd/check` is the only process that talks to Redis. The proxy, the dashboard, and `pkg/httplimit` are gRPC clients of `check.v1.Checker` over h2c. The call is `Check(key, cost)`. The answer is `allowed`, `remaining`, and `retry_after_ms`. The concurrency test fires 50 of those against a limit of 10, ten times, and expects exactly 10 allows. Go is not what makes that exact. One Lua `EVAL` reads the window, decides, and increments. A mutex in this process would not cover a second Check sharing the same Redis key.

### One algorithm, with a slot for the next

The algorithm sits behind `engine.Checker`. `internal/server` looks up a policy and calls `Check`. It does not import the limiter. `cmd/check` builds `slidingwindow.Limiter` and passes it in, so a second algorithm would not change the RPC. The one that ships is an approximate sliding window: Redis hash `rl:<key>` with fields `cs`, `cc`, and `pc`, and `PEXPIRE` of twice the window. Used is the previous window weighted by the time left in this one, plus the current count. `cost == 0` is a peek and does not create a key.

The seek and the update are one Lua script. `slidingwindow` embeds `script.lua` and runs it with `EVAL` on that hash. The script loads `cs`, `cc`, and `pc`, works out `used`, and only if `used + cost` still fits does it `HSET` the new counts and refresh the expiry. Redis runs that script to completion before another command on the same key, so two Checks cannot both read spare quota and both admit. A read in Go and a later write would race. A deny does not write.

Token bucket is not in the repo. Adding it means a second Lua path and a field on the policy so a rule can select it. Lookup already picks the rule: an exact `keys` entry, else the longest matching `prefixes` entry, else `default`. That choice sets `limit`, `window`, and `fail`. It does not set the algorithm. Choosing the algorithm per key or prefix is the follow-on, on that same lookup.

### Two ways in, one deny

Pattern A and Pattern B only differ in where that call runs. The proxy is `httputil.ReverseProxy` wrapped with the same `pkg/httplimit` middleware, so the 429 is not written twice and a deny never reaches origin. Allowed requests are forwarded by the standard library, which still drops hop-by-hop headers. Pattern B is that middleware on your own `net/http` handler. The demo is `cmd/origin -limited` on `:8000`. The key is an opaque string. Adapters take it from a header (`X-API-Key` here) or from the client IP. Check only looks the string up. A missing key is 400 `{"error":"bad_request"}`. A deny is 429 `{"error":"too_many_requests"}` with `X-RateLimit-Remaining` and `Retry-After`.

### The policy is a file

There is no app database. Counters are those Redis hashes, and a Redis restart can wipe them. The file Check loaded is the policy: `listen_addr`, `metrics_addr`, `redis.addr`, `default`, optional exact `keys`, optional `prefixes`. Check accepts YAML or JSON. The Compose file is `configs/compose/check.yaml`. `ListPolicies` returns every rule. `SetLimit` changes `limit` on a rule that already exists. It does not create a key, and it does not change `window` or `fail`. The next Check uses the new limit. The Redis counter is not reset.

The edit is one write at a time. `SetLimit` holds a mutex, clones the snapshot, writes a temp file in the same directory, syncs it, and renames it over the config. Only then does it store the new snapshot in the `atomic.Pointer` that `Lookup` reads. A failed rename leaves memory and disk as they were. Unknown targets and `limit <= 0` are rejected the same way. Checks do not take the write lock. They load the pointer. Compose bind-mounts the config directory, not a single file, because a rename cannot replace a file that is itself the mount.

### Redis down is the rule

Redis failure is a policy, and there are two layers. The rule’s `fail` is Check talking to Redis. `fail: open` allows the request with remaining and retry at 0. `fail: closed` denies it the same way. In the demo, prefix `free:` is fail-open at 20 per minute and `pro:` is fail-closed at 500 per minute. A store error latches `redis_up` down. While it is down, Check returns that fail result and does not call Redis. A later successful Check does not clear the latch. A background `PING` about once a second is what sets `redis_up` true again. If one Check has already been inside Redis for 50ms, later Checks fail immediately and trip the latch, so a stuck `EVAL` does not pile up. `/healthz` stays 200. Compose must not restart Check when Redis is stopped, or that demo goes away.

The other layer is the adapter when the Check RPC itself fails. `pkg/httplimit` defaults to closed: 429, remaining 0, no `Retry-After`, and the handler does not run. Open there means the origin runs. Killing Check is not the Compose demo. Killing Redis is.

### Counters from Check

The dashboard is a small page of counters. `cmd/dashboard` serves “Rate limit stats” and polls `GET /stats` every second, one `Stats` RPC. Each last-seen key shows `used`, `remaining`, `limit`, and `fail`: how much of the quota is spent, how much is left, the quota, and whether that rule is fail-open or fail-closed. The same snapshot has `redis_up`, which is Check’s view of Redis, not a ping the page makes itself. When that is false, the banner says fail-open keys still serve and fail-closed keys deny. At most 50 keys, held in the Check process. Restarting Check zeroes them. They are not stored in Redis.

The Limits form loads `GET /policies` and posts `POST /limits`, which is `SetLimit`: pick an existing rule, change its limit. No login and no WebSocket. The page does not export series of its own. Prometheus on `:9090` scrapes Check at `:2112` every second for `api_rate_limiter_requests_total`, `api_rate_limiter_blocked_total`, and `api_rate_limiter_redis_up`.

Multi-region, billing, and auth on the dashboard or `/metrics` are still out. The piece I want next is that second algorithm, chosen on the same key and prefix lookup, without a new way in.

<div class="shots">
  <img src="{{ '/assets/img/projects/api-rate-limiter/api-ratelimit.png' | relative_url }}" alt="Rate limit dashboard with Redis status and per-key counters">
</div>