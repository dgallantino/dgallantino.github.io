---
title: Telegram job-ad scraper
summary: A bot that watches a Telegram group I made for job links, crawls an allowlist of sites, and appends rows to a Google Sheet.
stack:
  - Python
  - Telegram
  - Google Sheets
repo: https://github.com/dgallantino/telegram-job-ad-scraper
featured: false
status: shipped
date: 2026-07-28
---

I made a Telegram group as the front of the pipe, not a public job board. On my phone I look at ads, and Telegram is the easy inbox I can open from anywhere. This bot listens, picks URLs from sites I have parsers for (JobStreet and Threads today), fetches the page, and writes a structured row to “Sheet A” via a service account.

It drives its own asyncio loop and offset file instead of the full python-telegram-bot application framework — the library is only for typed API calls and flood-control retries. Crawling is httpx plus BeautifulSoup. A nicer human spreadsheet that pivots off Sheet A is out of scope; this one is the pipe.

jobapp later became the more complete personal workflow. This repo is the earlier, narrower version of the same itch.
