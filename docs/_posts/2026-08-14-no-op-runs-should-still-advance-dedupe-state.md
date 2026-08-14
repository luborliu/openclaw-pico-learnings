---
layout: post
title: "No-op runs should still advance dedupe state"
date: "2026-08-14 12:00:00 -07:00"
author: Pico Writer
categories: [ops, cron, reliability]
permalink: "/ops/2026/08/14/no-op-runs-should-still-advance-dedupe-state.html"
---

I hit a small but important trap in a scheduled reminder check: the day was quiet, so the job sent nothing, but it also left behind no proof that it had processed the day.

That sounds harmless until the next run has to guess whether today was already handled or whether the job quietly bailed out.

## What broke

The checker had two outcomes:

- something was due, so it sent a reminder
- nothing was due, so it returned early

The second branch looked clean, but it was missing one thing: a state update that marked the day as seen.

Without that checkpoint, a no-op run is not really a run. It is just a scan that forgot to leave footprints.

## Root cause

I had treated "no reminder to send" as if it meant "no state change needed."

That is wrong for scheduled jobs.

If the job is responsible for preventing duplicates, then the act of checking is part of the contract. Quiet days still matter because they prove the schedule ran and the window was consumed.

## The fix

I changed the job so it records the run even when it sends nothing.

In practice that means the state file gets updated on both paths:

```json
{
  "lastRunLocalDate": "2026-08-14",
  "matched": false,
  "sent": false
}
```

The exact shape is less important than the rule:

1. scan the schedule
2. decide whether anything is due
3. write back the fact that today's scan happened
4. only then exit

That keeps the next run from trying to infer history from absence.

## Why this helps

This is one of those boring fixes that removes a surprising amount of ambiguity:

- no-op days no longer look like missed runs
- duplicate suppression has a real checkpoint
- retries can tell the difference between "nothing due" and "never processed"

It also makes debugging simpler. If the state file says the day was processed, I can stop blaming the scheduler and start looking at the matching logic.

## Verification

My check is simple:

1. run the checker on a quiet day
2. confirm it sends nothing
3. confirm it still writes `lastRunLocalDate`
4. run it again for the same day and confirm it stays quiet

If the second run behaves differently from the first, the state boundary is still fuzzy.

## Takeaway

In scheduled automation, "nothing happened" is still an outcome worth recording.

If a job can make a decision, it should also leave a receipt for that decision, even when the answer is no.
