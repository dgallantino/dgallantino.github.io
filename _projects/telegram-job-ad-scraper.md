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

I made a Telegram group as the front of the pipe, not a public job board. On my phone I look at ads, and Telegram is the inbox I can open from anywhere.

The bot listens in that group, keeps URLs from sites I have parsers for (JobStreet and Threads today), fetches the page, and appends a row to a Google Sheet.

A nicer spreadsheet on top of those rows is out of scope. jobapp later became the more complete personal workflow; this repo is the earlier, narrower version of the same itch.


## How it is built

Looking for a job kept splitting my attention. The part that needs my attention is the company, what they actually want, whether my experience helps them, and the cover letter. The part that does not is the bookkeeping: saving an ad I already liked, noting whether I applied, and where that application stands. A spreadsheet can hold that. This project is the pipe that fills it, so that attention stays on the work that needs a personal touch.

I also wanted it small and cheap. It runs on my own computer, and I did not want it eating the machine. The fast path was to use things I already had for free, and to keep this process in the background. At work I had wired Telegram bots to send alerts and to get machine or process status, then wait for the answer. This is the same shape. I drop a link in a group I made; the bot answers when it is done. Telegram was the easy front because I already knew it.

Google Sheets as the database was new to me. I did not want to build a screen just to look at rows, and I am the only person reading them, so a nicer display would have been wasted. The volume is low. What mattered was a place I can open from anywhere, without paying for a database.

There is no crawler here. I am not walking a site. I scrape one page, and only when I already know the layout. The adapters live in `job_scraper.scraper.sites`. The main program picks the one that matches the URL.

### When the process dies

The awkward part is picking up where it left off. A small local file stores the last Telegram offset, the `update_id`, so the next start has a reference for what was already ingested. I did not invent a separate id for each row. The sheet id is the Telegram chat id and message id joined together (`{chat_id}_{message_id}`). That coupling is the fallback. If the state file is corrupt or gone, the sheet can still say which message was last written. It is less accurate than the offset file. Telegram only keeps a short buffer of updates, so a long gap cannot be rebuilt from the sheet. For a personal pipe it is enough.

The sheet is written in two steps, on purpose. As soon as a message is ingested, the id and the URL go in. A supported site is marked `pending`. An unsupported site is marked `rejected`. I write the rejected rows anyway: the message id is then on record, and I can still open the link myself. A supported URL is handed to a worker. The worker marks the row `running`, and when the scrape finishes it becomes `finished` or `failed`. A successful scrape also fills the row with what it took from the page. On the next start, anything still `pending` or `running` is queued again, so a crash in the middle does not drop the link.