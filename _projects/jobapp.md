---
title: jobapp
summary: A personal job-ad scraper and cover-letter helper — one Go binary, SQLite, and an htmx UI on a home machine.
stack:
  - Go
  - SQLite
  - htmx
image: /assets/img/projects/jobapp/job_list-page.png
repo: https://github.com/dgallantino/jobapp
featured: true
status: shipped
date: 2026-09-10
---

jobapp is one Go binary on a home machine. It crawls sources I configure, pulls links from a Telegram group, and stores the ads in SQLite. The UI is templates and vendored htmx. There is no Node build.

The commands are `serve`, `crawl`, `telegram-check`, and `scrape-check`. `crawl` and `telegram-check` each do one pass and exit. systemd user timers run them. `serve` is socket-activated. Secrets stay in an XDG `.env`.

The UI is sign-in, sources, the ads, and a cover-letter draft. It does not apply for me, and it does not decide which jobs are worth it.

## How it is built

I do not want every job on a site. A crawl source is a search I already run, stored as a row: name, URL, and an adapter name. `crawl` walks the enabled rows, scrapes that page, and inserts ads whose `source_url` is not already in SQLite. A second pass skips what it already has. Threads, Instagram, and the Telegram marker are not crawl targets. Those rows exist so a link I sent can land in the same table.

`telegram-check` is the other door. I see a posting on my phone and drop it in the group. The command does one short poll of `getUpdates`, then exits. It is not a bot sitting in the group. A timer runs that pass, and another runs crawl. The last `update_id` lives in SQLite, in a one-row `telegram_state` table, so the next pass does not reread the same messages. Both are oneshot processes. Nothing in this pipeline is a long-lived worker.

### One adapter per site

The adapter is a column, not a plugin system. Built in today: `static` (`net/http` and goquery), JobStreet, Glints, Dealls, Kalibrr, Threads, and Instagram. A listing crawl stops around 100 jobs so one source cannot fill the database in a single pass. Detail pages are fetched alongside that, up to five at a time. `crawl` also waits a random 2–5 seconds between requests to the same host.

JobStreet, Dealls, and Kalibrr stay on HTTP. JobStreet is server-rendered cards and `rel=next`. Dealls and Kalibrr expose enough in page JSON or their own search API that a browser would be extra. Glints listings do not. Those need Chromium on the machine, via chromedp: scroll until 100 jobs, a login wall, or the card count stops growing. Signed out, that ceiling is often closer to 30. A Glints detail URL, and a link that arrives from Telegram, does not start a browser. If I ever put that browser in a container, I want Podman.

Threads and Instagram are the thin ones. The page is only a post, and empty fields can be filled by the same OpenRouter client the cover letters use. `scrape-check` runs any of these adapters against one URL and prints the result. It does not touch the database, which is how I look at a parser before I trust it with a source.

### The page has to stay light

The earlier pipe kept the rows in a Google Sheet, which I could open from anywhere. This database stays on the home machine. I use Tailscale so I can still reach the UI from anywhere, and the socket is bound to that address.

That link is a mesh VPN, not a fast local network, so a heavy page would feel slow. The pages stay mostly static: the server renders the list, and the script on the page is vanilla. It selects rows and copies a letter. The two things that are not on the first paint come through htmx. Opening a row fetches that job's description once and drops it inline. Generating a letter writes the draft into the page when the model finishes. There is no frontend build to ship across that link.

### It should not sit idle

I picked Go because I wanted a small binary, and because I wanted to write Go again. Past that binary, the language is not doing special work here. The refresh course was most of the reason for Go.

The same machine is where I do other work, and the memory on it is not spare. So jobapp runs as a systemd user unit on the host. I did not put the app in a container. `serve` should not wait around either: I am the only user, and I do not open the UI often. systemd listens on the socket and hands that listener to the process. The unit passes `-idle-timeout 30m`: half an hour with no HTTP, it finishes in-flight requests, closes SQLite, and exits. The next connection starts it again. Sessions live in that process, so I sign in again after an idle exit. That is an acceptable cost for not leaving it running.

Crawl and the Telegram check are timers I set from how much comes back. Crawl runs once a day at 20:00. The Telegram check runs every two hours. If a pass comes back thin, I lengthen that timer by hand. They are not a schedule the program insists on.

The database is `~/.local/share/jobapp/jobs.db`. Secrets are `~/.config/jobapp/.env`, mode 600: a bcrypt password, a session secret, the OpenRouter key, and the Telegram bot token plus chat id. Timers die on logout unless lingering is enabled for the user.

### The letter is a draft

An ad has a status I set myself: `new`, `applied`, `rejected`, or `ignored`. The profile the letter is written from is still a key/value stub: name, summary, work history, skills, tone, signature. I have not decided those need real structure yet. The prompt that turns that profile plus the ad into a letter is isolated in one function and still needs tuning. OpenRouter writes the draft. The row stores the text and which model produced it. The model does not change the status, and it does not choose the job.


<div class="shots">
  <img src="{{ '/assets/img/projects/jobapp/job_list-page.png' | relative_url }}" alt="Job ads table with status filters">
  <img src="{{ '/assets/img/projects/jobapp/job_detail-page.png' | relative_url }}" alt="Job detail with description and a cover letter draft">
  <img src="{{ '/assets/img/projects/jobapp/sources_list-page.png' | relative_url }}" alt="Crawl sources with adapter and enabled state">
</div>