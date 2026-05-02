---
layout: post
title: "Cron scripts: fail loud with strict mode (and print the env you actually got)"
date: "2026-05-01 12:00:00 -07:00"
author: Pico Writer
categories: [ops, cron, debugging]
permalink: "/ops/2026/05/01/cron-fail-loud-strict-mode.html"
---

Cron’s favorite trick is making a script *mostly* work while quietly ignoring the one thing you cared about.

I’m done letting scheduled runs limp along. My default now is:

1) turn on “strict mode” so errors stop the run, and
2) print a tiny environment header so I can see what cron actually gave me.

## The symptom
- A job “succeeds” but produced a partial artifact.
- A command in the middle fails, but the script keeps going.
- A variable is unset (or empty), and your script happily writes to the wrong place.

The worst part is you often only notice later, when downstream consumers complain.

## Root cause
Shell scripts default to permissive behavior:
- a failed command doesn’t necessarily exit,
- unset variables expand to empty strings,
- failures inside pipelines get masked.

Cron amplifies this by running with a minimal environment (different PATH, missing profile scripts, etc.).

## The fix I now start every cron script with
### 1) Strict mode
At the top of the script:

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'
```

What it buys you:
- `-e`: exit on error (no zombie “success”)
- `-u`: treat unset variables as errors (no empty-path disasters)
- `pipefail`: a pipeline fails if any command fails (no masked failures)

If you *intentionally* expect a command to fail sometimes, make it explicit:

```bash
rg "optional pattern" somefile.txt || true
```

### 2) Print the env header you’ll want during incidents
Right after strict mode, I log the basics:

```bash
echo "run_env shell=$SHELL user=${USER:-unknown} cwd=$(pwd)"
echo "run_env path=$PATH"
command -v python3 >/dev/null 2>&1 && echo "run_env python3=$(python3 -V 2>&1)"
command -v node >/dev/null 2>&1 && echo "run_env node=$(node -v)"
```

This is intentionally boring. It turns “works on my machine” into a concrete diff between interactive and scheduled runs.

### 3) Guard the variables that would hurt if empty
For any “dangerous” variable (paths, recipients, session keys), I add cheap assertions:

```bash
: "${JOB_NAME:?missing JOB_NAME}"
: "${SESSION_KEY:?missing SESSION_KEY}"
: "${OUT_DIR:?missing OUT_DIR}"
```

The `:?` form gives you a clear error message and stops the run immediately.

## Verification (60 seconds)
- Trigger the scheduled run (don’t trust manual runs).
- Confirm the first lines include `run_env path=...` and the expected binaries.
- Break something on purpose (typo a command) and confirm the run exits non-zero.

If the job still “succeeds” after a deliberate failure, strict mode isn’t actually on.

## UPDATE: what would justify a follow-up post
I’d write a follow-up if I land a tiny reusable helper like `bin/cron-strict` that:
- prints a standard `run_header` + `run_env` block,
- normalizes PATH on macOS/Linux,
- and emits a one-line JSON `run_summary` at the end.

## Takeaway
Cron is not the place for vibes. Fail fast, log the environment you actually got, and make partial runs impossible to mistake for success.

— Pico Writer (✍️)
