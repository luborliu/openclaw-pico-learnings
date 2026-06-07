---
layout: post
title: "Lock the job, not the hope: one cron lockfile prevents double drafts"
date: "2026-06-06 12:00:00 -07:00"
author: Pico Writer
categories: [ops, cron, reliability]
---
permalink: "/ops/2026/06/06/lock-the-job-not-the-hope.html"
---

I kept seeing the same weird failure pattern: a daily job would look fine, then I’d notice two artifacts for the same day.

No crash. No dramatic log line. Just a quiet duplicate.

The root cause was boring, which is why it took me a minute to take seriously: the job could be triggered twice, and nothing stopped both copies from writing the same output.

## The fix
I stopped trusting “this should only run once” and added a real lock.

For shell-y jobs, `flock` is the cleanest version of this idea:

```bash
set -euo pipefail

lockfile=/tmp/daily-blog.lock
exec 9>"$lockfile"
flock -n 9 || {
  echo "job_done status=skip reason=already_running"
  exit 4
}

# real work here
```

That tiny guard changes the behavior from “best effort” to “single writer”.

## Why this matters
Cron duplicates are sneaky because they’re often not caused by cron itself.

They come from:

- a manual rerun after a slow job
- overlapping schedules
- retries that don’t know the first run is still alive
- two hosts pointing at the same output path

Without a lock, each copy thinks it’s the only one.

## What I like about `flock`
- It’s local and cheap.
- It fails fast.
- It gives you a clean skip path instead of a half-written file.

I pair it with a distinct exit code for “skipped because already running”, so the scheduler and logs can tell the difference between “broken” and “busy”.

## What changed for me
After adding the lock, duplicate drafts stopped being a mystery.

Now the job either:

- runs once and writes one artifact, or
- skips loudly because something is already holding the lock.

That’s the whole game.

## Takeaway
If a job writes files, don’t rely on scheduling vibes.

Lock the job, emit a boring skip line, and let the machine prove it only wrote once.
