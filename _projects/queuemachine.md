---
title: Queue Machine
summary: A clinic queue system with a kiosk, a staff desk, and a public board — one Django deploy, several branches.
stack:
  - Python
  - Django
  - HTMX
image: /assets/img/projects/queuemachine/ticket-machine.png
repo: https://github.com/dgallantino/queuemachine
featured: true
status: shipped
date: 2026-07-06
---

Queue Machine is a Django app that runs the ticket queue. It has three surfaces: a kiosk where a customer taps a service and a paper ticket prints, a manager dashboard where the front desk calls the next number, and a board in the waiting area showing what each counter is serving.

One deployment covers three branches. A staff account belongs to one or more organizations and picks one at sign-in, and every list, form, and queue action after that is scoped to it. Numbers restart per service, per branch, per day.

The call is also spoken. The server stitches a WAV out of pre-generated fragments and the manager's own browser plays it, so the speaker sits at the desk instead of next to the server. It runs on the clinic's self-hosted machine as Docker Compose: nginx in front, Gunicorn running Django, MySQL behind it.

<div class="shots">
  <img src="{{ '/assets/img/projects/queuemachine/ticket-machine.png' | relative_url }}" alt="Ticket machine issuing a queue number">
  <img src="{{ '/assets/img/projects/queuemachine/manager.png' | relative_url }}" alt="Staff queue manager dashboard">
  <img src="{{ '/assets/img/projects/queuemachine/display.png' | relative_url }}" alt="Public information board showing current queues">
</div>

## How it is built

This started as an internship. I was the one-man IT department at a beauty clinic, and almost every week something in the old queue system needed me. The service ran on one admin's desktop, so every fix stopped that person from working. It had been built when there was one branch, then jerry-rigged so the other branches could reach it. Two surfaces: a machine that prints the ticket, and an admin screen that calls it. What I built replaced that, added a third surface for the waiting area, and changed where the voice comes out.

### The number is issued in one place

A queue row can exist before it has a number. A booking is created in the morning by staff; a walk-in is created the moment someone taps a service. Neither is numbered on creation. The number is assigned when the ticket is actually printed, so the order on paper is the order people arrived at the kiosk, not the order rows landed in the table.

That assignment is the one operation I did not want to be clever about. It runs in a transaction, takes a row lock on the queue and on the service, reads the highest number printed today for that service, and adds one. It is idempotent: if the row already has a number and is marked printed, it returns it unchanged rather than burning a second number on a retry. Moving a queue to another service goes through the same path — it clears the called and finished flags, drops the counter assignment, stamps a new print time, and takes a fresh number in the target service.

The kiosk itself is dumber than it looks, and that was the point. The hardware was already standing there — a computer, a touchscreen, and a printer wired together — and the operating system already knew how to talk to all three. Anything I wrote to manage the printer directly would have been me re-implementing something that worked. So tapping a service posts a form, gets the rendered ticket HTML back, writes it into an offscreen iframe, and calls print on that frame. No PDF, no print server, no driver work. Whatever printer the browser is pointed at is the ticket printer.

### Three clinics on one deployment

The old system leaked across branches because it was never designed to have more than one. This one treats the branch as a boundary and checks it in three places, which is more than strictly necessary and on purpose.

The session holds the chosen organization, and every request re-checks that the signed-in user is actually a member of it rather than trusting the session value. Querysets for queues, services, counters, and customers are all filtered through the organization. Forms narrow their foreign-key choices to that organization and reject a submitted id from outside it. Then the service layer asserts it again before writing: a queue must belong to the org, a counter must belong to the same org as the queue's service, a move target must too.

Ids in URLs are UUIDs, not sequential integers, so there is nothing to walk. But the failure I was protecting against is boring and human — a stale tab, a shared machine, someone switching branches mid-shift — and the cost of a wrong answer is someone at one branch's desk hearing a number that belongs to another.

### The voice is stitched, not spoken live

This is the part that actually justified the rewrite. In the old system the speaker was plugged into the server machine, which meant the spoken call only worked at the branch where the server physically lived. Everyone else got a silent screen.

My first version was not much better. Calling a queue hit Google Translate's TTS endpoint live, per call, and streamed the MP3 back. It worked on a good day, it added latency on a bad one, and it broke entirely when the token scheme changed — there is still a monkeypatch in the codebase for exactly that.

So the audio moved to build time. A management command reads the database and writes a sound map: the fixed phrases, the digits and number words for the language, the queue letters that services actually use, and the spoken name of every counter booth that exists. A second command walks that map and generates one WAV per fragment through gTTS, retrying with exponential backoff and jitter, then converting each MP3 to mono 22,050 Hz WAV with ffmpeg. Fragments that already exist are skipped, so re-running after adding a counter only generates the new one.

Calling a queue at runtime touches no network. The number is decomposed into fragment keys and the files are concatenated in order. Indonesian gets its own decomposition — ones below ten, `sepuluh` and `sebelas` as special cases, teens as *ones + belas*, tens plus ones, `seratus` at a hundred, and *ones + ratus +* the remainder recursively above that. English has a separate branch for the same reason: the word order is not the same and a shared "just split the digits" rule produces something nobody would say out loud. The ceiling is 999.

Concatenation is deliberately strict. It reads each WAV's channels, sample width, frame rate, and compression, and refuses to join anything that does not match instead of writing a file that plays as noise. That strictness is why generation normalizes every fragment through the same ffmpeg settings rather than trusting whatever came back from the API.

Playback is the last piece and the one that fixed the original complaint. The audio element on each manager row preloads nothing and points at the compose endpoint. Clicking the bell posts the call and marks the row; clicking it again on an already-called row plays or pauses the sound. The audio comes out of the machine the admin is sitting at. Every branch gets a voice, and the server does not need a speaker at all.

### Polling, because the numbers said so

The board refreshes each counter card every five seconds and the waiting list every ten, both over HTMX. The manager table loads its rows when it scrolls into view and has a manual refresh button. The kiosk's booking sidebar checks for new bookings every five minutes and appends only what is newer than the last row it has. There are no websockets, no channels layer, no background worker. The whole thing is request/response.

I pulled the production ticket table before writing this to check whether that was still defensible. Between 28 December 2025 and 28 September 2026 — 275 calendar days, 244 of which issued at least one ticket — the three branches issued 1,464 tickets. The busiest single day across all three was 28. The busiest single service at a single branch on a single day was 19.

The weekday breakdown was the part I did not expect to be so sharp. One of the smaller branches issues nearly everything on Mondays, Thursdays, and Saturdays, with a handful of stray tickets on other days. The other is almost entirely Wednesday and Friday. The central branch runs every day of the week, Sundays included, and takes more than the other two combined. The branches are sharing staff on a rota, and the queue data shows the rota clearly. Within a day the load stacks up between 14:00 and 16:00.

So the 999 ceiling on spoken numbers is not a ceiling anyone will reach, and five-second polling from a handful of screens is not load. Nothing here was ever a scaling problem. It was a correctness problem across branches, and a plumbing problem about which speaker the sound comes out of.

### What is still old

The first commit is from 2018, and the migration history still carries every step from that year. The 2026 pass rewrote the frontend, containerized the deployment, moved queue logic out of views and forms into a service layer, and replaced the live TTS call with the fragment pipeline. Bootstrap, jQuery, Font Awesome, and jQuery UI are gone; Tailwind, HTMX, Alpine, Lucide, Tom Select, and Flatpickr are pinned and fetched at build time rather than committed, so there is no vendored blob in the repository. The tests cover the parts I was least willing to guess at: number decomposition in both languages, the compose recipe, the organization scoping, and the retry behaviour.

The live TTS endpoint is still in there next to the composed one. I left it as a fallback and never removed it.

Booking is the honest gap. There is a booking flag on the queue, a form to create one, and a sidebar on the kiosk to print bookings that have arrived. In nine months of production data, not one row has that flag set. Staff made a service called "BOOKING" and use that instead — ten tickets, all of them ordinary walk-up tickets with a different label on them. The feature I built and the workflow they adopted are not the same thing, and I only learned that from the export while writing this.
