---
title: API rate limiter Python SDK
summary: Wraps a FastAPI or Flask app so each request is checked against the rate-limiter gRPC service before origin work runs.
stack:
  - Python
  - gRPC
  - FastAPI
  - Flask
repo: https://github.com/dgallantino/api-rate-limiter-python-sdk
featured: false
status: shipped
date: 2026-09-22
---

Pattern B on the [API rate limiter]({{ '/work/api-rate-limiter/' | relative_url }}) service is Go `net/http` middleware. The apps I drop a limiter into are Flask and FastAPI, so the view needs its own client and should not run on a deny.

`RateLimitMiddleware` sits in front of any ASGI app, and `RateLimitWSGI` sits in front of any WSGI app. You do not need FastAPI or Flask to use them. Each wrapper is configured once for the whole app: who the caller is (an API-key header, or the client IP), how much the request costs, and what to do when Check cannot answer. A missing key is rejected immediately. If Check is down, the default is to deny the request so it never reaches your app. You can switch that to allow. That choice only covers a failed Check call. When Check does answer, the app follows that answer, including a decision Check made because Redis was down.

FastAPI and Flask have smaller hooks for the same check. On FastAPI, one covers every request and another can be attached to a single route. On Flask, the hook runs before each view. To cover a whole Flask app the same way as the generic wrapper, put `RateLimitWSGI` around it. The async client has to be created on the same event loop that serves the app. The regular client used with WSGI does not.

`make proto` pulls `proto/` from tag `v0.1.0` of the service and writes stubs into `gen/`. Neither directory is committed. This repo does not run Redis or the proxy.
