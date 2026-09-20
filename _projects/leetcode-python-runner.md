---
title: LeetCode Python runner
summary: Edit a LeetCode-style solution locally and run it against YAML cases without the website.
stack:
  - Python
repo: https://github.com/dgallantino/leetcode-python-runner
featured: false
status: shipped
date: 2026-08-02
---

The website is a poor editor. This CLI looks for `problems/<name>/solution.py` plus `cases.yaml`, instantiates `Solution`, and calls the method named in the YAML. Cases can set timeouts and choose exact versus unordered compares.

That is all I wanted: same class shape as LeetCode, tests next to the file, Python 3.10+. It is a practice harness, not a scraper for the platform.
