---
title: jobapp
summary: A personal job-ad scraper and cover-letter helper — one Go binary, SQLite, and an htmx UI on a home machine.
stack:
  - Go
  - SQLite
  - htmx
repo: https://github.com/dgallantino/jobapp
featured: true
status: shipped
date: 2026-09-10
---

I got tired of copy-pasting listings into a spreadsheet. jobapp is a single-user tool that crawls sources I configure, can poll Telegram for new ads, and keeps everything in SQLite with a small web UI.

I do not want every job on a site. Crawl sources are the searches I already run, so the database only fills with listings I would have looked at anyway. `telegram-check` is the other path: I see a posting on my phone, and I want it in the same SQLite file. A systemd timer short-polls Telegram. It is not a bot sitting in the group all day.

The binary is the product: `serve`, `crawl`, `telegram-check`, or scrape one URL to stdout. No Node build. htmx is vendored. It lives on a home machine (I use Tailscale), with systemd user units, a socket, and timers rather than a public cloud app. Socket activation is because I am the only user and I do not open it often — nothing needs to sit idle waiting for me. Secrets stay in an XDG `.env`.

The UI is for me: sign in, add sources, read ads, and generate a cover letter when I have already decided to apply. It is not an auto-apply system. A model can draft the letter. It does not pick which jobs are good.
