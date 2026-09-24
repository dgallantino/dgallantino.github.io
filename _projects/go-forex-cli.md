---
title: Forex CLI
summary: A personal command that converts an amount across currencies and keeps rates in a local cache.
stack:
  - Go
repo: https://github.com/dgallantino/go-forex-cli
featured: true
status: shipped
date: 2026-08-20
---

I wanted `forex-rate 10 usd idr jpy` on a machine without opening a browser. The CLI talks to ExchangeRate-API, then writes a JSON cache under XDG so later conversions stay snappy. Config is a small TOML file, not a project `.env`.

Realtime ticks are not the point. A week-long cache is the default; `--refresh` fetches again. There is an optional baked-in rate for a personal unit I use in notes. That is the whole product, on purpose.


## indepth notes

this 
