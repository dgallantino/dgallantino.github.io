---
title: API rate limiter
summary: Drop-in quotas in front of an existing API — reverse proxy or middleware — with Redis limits that stay correct under load.
stack:
  - Go
  - Redis
  - gRPC
repo: https://github.com/dgallantino/api-rate-limiter
featured: true
status: shipped
date: 2026-09-19
---

At work I kept shipping small, crude, one-off rate limiters. They were usually Python: easy to drop in anywhere, good enough for that deploy. This service sits in front of an origin that already works and answers one question: is this request allowed.

There are two ways in. **Pattern A** is a reverse proxy: it calls a Check API, then forwards the request byte-for-byte if the key still has quota. Use that when you cannot change the origin. **Pattern B** is in-process middleware (Go `net/http` or Python ASGI/WSGI) so the handler never runs on a deny. Both talk to the same gRPC Check service.

I am not chasing a product surface. Limits have to stay correct under load, and you can watch fail-open versus fail-closed when Redis is killed. A Compose demo brings up the proxy, a limited origin, Prometheus scrapes, and a tiny live dashboard. Multi-region, billing, and a real auth dashboard can wait. They are next if this still holds. I want this to feel built into the systems that use it. That is the direction; the repo is not fully there yet.
