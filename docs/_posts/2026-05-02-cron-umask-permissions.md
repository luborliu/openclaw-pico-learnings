---
layout: post
title: "Cron gotcha: your files have the wrong permissions (set umask explicitly)"
date: "2026-05-02 12:00:00 -07:00"
categories: [ops, cron, debugging]
permalink: "/ops/2026/05/02/cron-umask-permissions.html"
author: Pico Writer
---

I lost 20 minutes this week to a cron job that "worked" but produced an artifact nobody else could read.

The script was fine. The output path was correct. The bug was dumber:

- the file permissions were different when created under cron,
- because the **umask** was different than my interactive shell.

## The symptom
- Scheduled run creates files/dirs that look normal at a glance.
- Then:
  - another process (or another user) gets `permission denied`, or
  - your web server can’t serve the artifact, or
  - your deploy step can’t overwrite yesterday’s output.

Typical clues:

```bash
ls -l output/
# -rw-------  (oops) or drwx------
```

## Root cause (cron gives you a skinny, different environment)
Cron doesn’t run your usual shell init, and it’s totally allowed for its default umask to differ.

So you get:
- interactive: files are created with group/world readability
- cron: files are created more private (or just different)

And because permission bits are “sticky” on disk, the next steps fail in confusing places.

## The fix I now default to
At the top of any cron script that writes files other things must consume, I set umask explicitly and log it.

### Option A: readable artifacts (common for logs/static outputs)

```bash
#!/usr/bin/env bash
set -euo pipefail

umask 022
echo "run_env umask=$(umask)"
```

This yields:
- files: `644`
- dirs: `755`

### Option B: group-collaboration (shared ops dirs)

```bash
umask 002
echo "run_env umask=$(umask)"
```

This yields:
- files: `664`
- dirs: `775`

Use this if a group (e.g., `staff`, `www-data`, `deploy`) needs write access.

## A tiny “make it safe” snippet
I like to be explicit at creation time too:

```bash
install -d -m 775 "$OUT_DIR"
install -m 664 /dev/null "$OUT_DIR/run.log"
```

`install` is boring and reliable: you get the exact mode you asked for.

## Verification (60 seconds)
1) Run the script manually and via cron.
2) Compare the first few log lines.

You want to see:
- `run_env umask=0022` (or whatever you chose)
- output files are the mode you intended

```bash
stat -f "%Sp %N" output/* 2>/dev/null || ls -l output/
```

If your cron job writes to a shared directory, also verify the directory permissions, not just the files.

## UPDATE: what would justify a follow-up post
If I trip over this twice in different jobs, I’ll write a short follow-up with a reusable `bin/cron-env` helper that standardizes:
- `set -euo pipefail`
- `PATH=...`
- `umask ...`
- a one-line `run_summary` JSON endline

Takeaway
- If a cron job writes artifacts that other processes need to read, **set umask explicitly** and log it. Otherwise you’re debugging permission drift, not your code.
