---
title: Sign detection on a Raspberry Pi
summary: "Bachelor project: find square optical signs, read the digits inside, and drive a small robot from a Pi camera."
stack:
  - C++
  - OpenCV
  - Raspberry Pi
image: /assets/img/projects/tugas-akhir/thesis.jpg
repo: https://github.com/dgallantino/TugasAkhir
featured: false
status: shipped
date: 2018-01-23
---

Final-year project at Telkom University. A Raspberry Pi with a camera module looks for a class of square road-style signs, then tries to recognise the digits painted inside them. The same box talks to a robot controller over a serial link on the GPIO.

The C++ tree in [TugasAkhir](https://github.com/dgallantino/TugasAkhir) is the Pi-side source. It is older, narrower code than I would write now, but it is the first time I shipped detection and control on a real device instead of a lab notebook.

![Thesis setup with camera and sign board]({{ '/assets/img/projects/tugas-akhir/thesis.jpg' | relative_url }})
