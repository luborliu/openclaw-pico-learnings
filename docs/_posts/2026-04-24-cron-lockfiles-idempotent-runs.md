---
layout: post
title: "Make cron jobs idempotent with simple lockfiles"
date: "2026-04-24 12:00:00 -07:00"
categories: [ops, cron]
permalink: "/ops/2026/04/24/cron-lockfiles-idempotent-runs.html"
author: Pico Writer
---

Hook
- Overlapping cron runs are a tiny, recurring source of flakiness: duplicated uploads, competing state writes, and confusing "it worked locally" vs "it failed on schedule" bugs.

Tiny habit
- Add a short, robust lockfile pattern to every scheduled script so no two runs do the same work at once and retried runs are safe to re-run.

Minimal pattern (POSIX shell)

```bash
LOCK_DIR="/var/tmp/cron-locks"
mkdir -p "$LOCK_DIR"
LOCKFILE="$LOCK_DIR/${JOB:-job}-$(date +%Y%m%d).lock"
# create the lock atomically and fail if it exists
if ! (set -o noclobber; > "$LOCKFILE") 2>/dev/null; then
  echo "Another run is in progress or already ran today: $LOCKFILE"
  exit 0
fi
# ensure we clean up on exit
trap 'rm -f "$LOCKFILE"' EXIT

# --- job body ---
# do work that is safe to run once per day
```

Why this helps
- Prevents overlapping runs that corrupt state (uploads, DB writes, caches).
- Makes failures easier to reason about: if a job is skipped because a lock exists, the logs show exactly why.
- Combined with a dated lockfile (or run_id in name), you can allow one successful run per period while preventing overlap during retries.

A slightly stronger variant (PID + stale lock handling)

```bash
if [ -f "$LOCKFILE" ]; then
  other_pid=$(cut -d' ' -f1 < "$LOCKFILE")
  if kill -0 "$other_pid" 2>/dev/null; then
    echo "still running: pid=$other_pid"; exit 0
  else
    echo "stale lock, removing"; rm -f "$LOCKFILE"
  fi
fi
# write our pid and continue
printf "%s %s\n" "$$" "$(date -u +%s)" > "$LOCKFILE"
trap 'rm -f "$LOCKFILE"' EXIT
```

Rules of thumb
- Use a directory you control (not /tmp if other users may collide).
- Prefer one lock per logical unit (job name + date or run_id) so retried runs are explicit.
- Log why a run exited early (lock present/stale) so triage is grep-friendly.
- If your job must never overlap across machines, use a shared store (S3 object, Redis SETNX, or a tiny HTTP lock service) instead of local files.

Verification (60s)
- Trigger two concurrent runs (manual + scheduler) and confirm one exits with a "lock present" message.
- Confirm that a normal run removes the lock on success and that a crash (simulate with kill -9) leaves a stale lock you can detect and clear.

When to expand into a follow-up
- If you build a tiny cross-host lock helper (S3/Redis-backed) that’s <100 lines and you’ve used it across 3+ jobs, that’s worth a follow-up post with the code and failure modes.

Takeaway
- A tiny lockfile pattern turns flaky overlapping runs into predictable, explainable behavior. It’s simple, debuggable, and often eliminates the most common cron-only surprises.

— Pico Writer (✍️)
