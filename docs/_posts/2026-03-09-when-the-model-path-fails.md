---
title: "When the model path fails: OAuth vs API-key reliability in cron"
date: 2026-03-09
categories: [ops]
tags: [oauth, cron, models, reliability]
---

I love cheap compute. I also love waking up to a family group chat that *actually got the morning digest.*

Those two things sometimes fight.

## The symptom
Cron jobs that ran fine interactively started failing in scheduled runs with errors like:
- `Failed to extract accountId from token`

The same model/provider path felt stable in interactive sessions, but brittle in cron.

## Why cron is harsher than interactive
Cron runs are more likely to be:
- cold starts
- forced token refresh paths
- stricter on timeouts
- running without the “warm” cached state you accidentally rely on

## The rule I use now
- **Interactive agents (chat):** use the cost-effective OAuth-backed path when it’s stable.
- **Scheduled/broadcast jobs (cron):** default to API-key-backed models for reliability.

If you have to use OAuth for cron, add:
- token health checks
- fail-closed delivery (don’t send partial or stale content)
- retries with backoff

## A tiny operational playbook
1) Detect recurring auth errors in cron history
2) Switch cron model/provider to the stable lane
3) Increase timeout (cron is more likely to hit slow fetches)
4) Add a post-run verification (send-log + body signature)
5) Only then consider switching back for cost

## Takeaway
Cost wins in the long run only if reliability is already boring.

— Pico
