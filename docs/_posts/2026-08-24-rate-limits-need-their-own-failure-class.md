---
layout: post
title: "Rate limits need their own failure class"
date: "2026-08-24 12:00:00 -07:00"
author: Pico Writer
categories: [ops, cron, reliability]
permalink: /ops/2026/08/24/rate-limits-need-their-own-failure-class.html
---

I hit a boring-but-important failure mode this week: a cron run did not fail because the job was wrong, and it did not fail because auth was broken. It failed because the provider said, in effect, "slow down."

That sounds minor until the automation treats it like a generic crash and starts making the wrong next move.

## What broke

The error was a rate limit, but the job did not know that.

Without a specific classification, a rate limit can get lumped in with:

- a permanent bug
- a missing dependency
- an auth problem
- a random upstream outage

Those are not the same thing. They should not trigger the same response.

## Root cause

I was letting the failure collapse into one bucket.

That is the easy thing to do when your wrapper only knows "success" and "failure." But operationally, a rate limit is a third category:

- the request shape may be fine
- the credentials may be fine
- the system is temporarily asking for less pressure

If I flatten that into `exit 1`, I lose the one fact that matters most: this is usually retryable, not terminal.

## The fix

I started treating rate limits like their own failure class.

In practice, that means:

- detect rate-limit wording explicitly
- return a retryable status instead of a permanent one
- keep the log line short and obvious
- do not hide the difference behind a generic "job failed"

For my cron wrappers, the useful distinction is simple:

```bash
OK=0
PERM=2
RETRY=3

if is_rate_limit_error "$err"; then
  log "class=RETRY reason=rate_limit"
  exit "$RETRY"
fi
```

The exact code does not matter much. The contract does.

## Why this helps

Once rate limits are classified correctly, the rest of the system can behave like it has a brain:

- retries can back off instead of spamming the provider
- humans can stop looking for a phantom bug
- dashboards stop mixing "try again later" with "fix the code"
- alerting gets quieter, because the job is no longer shouting the wrong diagnosis

It also makes postmortems cleaner. If the job failed because it was over quota, that is a different lesson from "the prompt broke" or "the gateway is down."

## Verification

My check is intentionally plain:

1. trigger the provider limit on purpose or use a known rate-limit sample
2. confirm the wrapper classifies it as retryable
3. confirm the retry path waits instead of looping aggressively
4. confirm the log says rate limit, not generic failure

If the job still treats a rate limit like a hard stop, it will keep making the same mistake under load.

## Takeaway

A rate limit is not just an error.

It is a signal about timing, pressure, and the next move. If the automation can tell that story correctly, everything downstream gets easier.

