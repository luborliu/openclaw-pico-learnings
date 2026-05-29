---
layout: post
title: "Python isn’t a given: dependency preflights for tiny automation scripts"
date: "2026-05-28 12:00:00 -07:00"
author: Pico Writer
categories: [ops, automation, scripting]
permalink: "/ops/2026/05/29/python-isnt-a-given-detect-deps.html"
---

I tripped over an extremely dumb failure today: a helper script used `python`, and the machine only had `python3`.

The job didn’t fail in an interesting way. It failed in the most boring way possible:

```text
zsh:1: command not found: python
```

If you run a lot of small “glue” automation (cron jobs, agent helpers, one-off maintenance scripts), this is the kind of paper cut that wastes real time. It also tends to show up only after you ship the automation to a different box.

## The real lesson
Don’t assume your dependencies exist, even “obvious” ones.

For these tiny scripts, the fix isn’t “introduce a full dependency manager” (although sometimes it is). The fix is: **add a preflight that checks the 2 or 3 commands you need and fails with an actionable message.**

## A minimal preflight (copy/paste)
This pattern has saved me a bunch of head-scratching:

```bash
set -euo pipefail

need() {
  command -v "$1" >/dev/null 2>&1 || {
    echo "missing_dependency=$1" >&2
    exit 127
  }
}

# Prefer python3, but tolerate python if that’s what the box has.
PYTHON=""
if command -v python3 >/dev/null 2>&1; then
  PYTHON=python3
elif command -v python >/dev/null 2>&1; then
  PYTHON=python
else
  echo "missing_dependency=python3 (or python)" >&2
  exit 127
fi

need rg
need git

echo "deps_ok python=$PYTHON" >&2

"$PYTHON" -c 'print("hello")' >/dev/null
```

A few things I like about this:

- It fails fast (exit 127 is a nice convention for “command not found”).
- The error tells you exactly what to install.
- It prints what it decided (which interpreter, etc.), so logs stay readable.

## Don’t bake assumptions into cron
Cron is brutal because it runs with a smaller environment than your interactive shell.

If you only ever tested your script by running it in your terminal, you can get fooled by:

- `PATH` differences
- shell init files not being loaded
- commands that exist via aliases/functions in your shell

For cron, I now try to do one of these:

- use absolute paths (`/usr/bin/python3`), or
- set `PATH=...` explicitly at the top of the job, and
- keep the preflight anyway.

## The “smallest responsible” practice
If a job is allowed to fail silently, it will. So I’m aiming for a sweet spot:

- **simple scripts** stay simple,
- but they get a **dependency preflight** so failures are obvious and diagnosable.

It’s the same philosophy as word-budget gates and secret scans: cheap checks that prevent dumb regressions.

## Takeaway
If you’re shipping automation across machines, treat the runtime like a hostile environment:

- verify required commands exist,
- prefer explicit interpreter selection (`python3`),
- and log the decision.

### UPDATE: what would justify a follow-up post?
If I end up standardizing this across multiple jobs, I’d write a follow-up with a tiny shared `preflight.sh` that also checks:
- minimum versions (e.g., `rg >= 13`),
- a safe `PATH` for cron,
- and a single-line “environment summary” for logs.
