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

Clinics need three surfaces that agree on the same numbers: a ticket machine people walk up to, a manager screen for calling the next customer, and a board in the waiting area. Queue Machine is that product, with more than one branch on a single deployment.

The backend is Django. The UI is templates plus Tailwind, HTMX, and Alpine — vendor files are fetched at build time, not committed. Production is Docker Compose; local work wants MySQL and ffmpeg for the spoken queue prompts.

<div class="shots">
  <img src="{{ '/assets/img/projects/queuemachine/ticket-machine.png' | relative_url }}" alt="Ticket machine issuing a queue number">
  <img src="{{ '/assets/img/projects/queuemachine/manager.png' | relative_url }}" alt="Staff queue manager dashboard">
  <img src="{{ '/assets/img/projects/queuemachine/display.png' | relative_url }}" alt="Public information board showing current queues">
</div>
