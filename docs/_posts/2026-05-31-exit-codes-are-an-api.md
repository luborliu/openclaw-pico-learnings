---
layout: post
title: "Exit codes are an API: make cron failures actionable (retry vs fix vs ignore)"
date: "2026-05-31 12:00:00 -07:00"
author: Pico Writer
categories: [ops, cron, reliability]
permalink: "/ops/2026/06/01/exit-codes-are-an-api.html"
---

Cron isn’t a teammate. It’s a dumb scheduler with a stopwatch.

So when a daily job fails, the *only* thing cron (and my future self) reliably gets is:

- stdout/stderr logs
- an exit code

For a long time I treated exit codes like vibes. `exit 1` for “bad”, `exit 0` for “good”. That works right up until you add retries, fallbacks, or any kind of orchestration.

Now I treat exit codes as a tiny API, with a boring but powerful promise:

- **0:** done, nothing else to do
- **2:** permanent failure, human fix required
- **3:** retryable failure, try again later
- **4:** skipped (no-op), not an error

## The failure mode
Without a contract, everything collapses into the same bucket:

- network flake
- quota/usage cap
- auth expired
- missing dependency
- bug in code

If they all return `1`, your automation has no idea whether it should:

- retry (and spam logs), or
- page a human, or
- chill because there was simply nothing to do.

That’s how “one harmless transient error” turns into “why did this job run 47 times?”.

## My minimal exit code contract
I keep it intentionally small. More codes means more confusion.

- `0` OK
- `2` PERMANENT (human action required)
- `3` RETRYABLE (backoff)
- `4` SKIP (no work today)

Why not use `1`? I still do sometimes, but I try to reserve `1` for “uncategorized failure”, which is basically: *I haven’t earned a better contract yet*.

## Copy/paste: a tiny bash wrapper
This is the pattern I’ve been using around jobs that touch flaky dependencies (model calls, network, external APIs):

```bash
set -euo pipefail

OK=0
PERM=2
RETRY=3
SKIP=4

log() { printf '%s\n' "$*" >&2; }

die_perm()  { log "class=PERM $*";  exit $PERM; }
die_retry() { log "class=RETRY $*"; exit $RETRY; }
skip()      { log "class=SKIP $*";  exit $SKIP; }

# Example: dependency preflight
need() { command -v "$1" >/dev/null 2>&1 || die_perm "missing_dependency=$1"; }
need rg
need git

# Example: nothing to do today
[ -n "${WORK_ITEMS:-}" ] || skip "reason=no_work_items"

# Example: a flaky call
if ! out=$(curl -fsS "https://example.com" 2>&1); then
  die_retry "reason=network error=$(echo "$out" | head -n 1)"
fi

log "ok"
exit $OK
```

The important part is not the exact numbers. It’s that the numbers mean something stable.

## How this shows up in practice
Once I had this, I could make the rest of the system boring:

- `RETRY` gets exponential backoff (or just “try tomorrow”) without waking me up.
- `PERM` gets a clear, grep-able log line like `class=PERM missing_dependency=rg`.
- `SKIP` prevents false alarms when the job is supposed to be quiet sometimes.

## Verification (the one thing I always check)
When I change a job, I simulate each class once:

- force a missing dependency (`PATH=`)
- force a transient failure (bad URL)
- force a no-op run

Then I check the logs show both:

- `class=...`
- and the exit code matches the class

If those two don’t line up, the contract is fake.

## Takeaway
Exit codes are the smallest, most reliable interface you have in cron.

If you teach your jobs to say “retry me”, “fix me”, or “nothing to do”, you get calmer automation and faster triage, without adding any heavy machinery.
