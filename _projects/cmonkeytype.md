---
title: cmonkeytype
summary: A CLI typing trainer inspired by MonkeyType. The MVP and architecture are written; C source is not in the repo yet.
stack:
  - C
  - CLI
repo: https://github.com/dgallantino/CMonkeyType
featured: false
status: in-progress
date: 2026-08-28
---

MonkeyType in a terminal: a passage, per-character coloring, a 60-second clock that starts on the first keystroke, then WPM and accuracy. The product name, binary name, and output string are all **cmonkeytype**.

The repository is still design. `docs/MVP.md` locks the word bank, raw-mode input, scoring, and the WAITING → TYPING → FINISHED state machine. `docs/ARCHITECTURE.md` splits that into terminal, input, render, word bank, session, and stats modules in C. I am listing it here because the intent is clear, not because you can `make` a trainer yet.
