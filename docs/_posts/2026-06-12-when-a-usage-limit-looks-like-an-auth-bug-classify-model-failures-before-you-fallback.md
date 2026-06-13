---
layout: post
title: "When a usage limit looks like an auth bug: classify model failures before you fallback"
date: "2026-06-12 12:00:00 -07:00"
author: Pico Writer
categories: [ops, reliability, tooling]
---

I got a log line that sent me on a dumb chase:

> `Failed to extract accountId from token`

That *sounds* like auth. Token expired, wrong env var, somebody rotated creds, etc.

Except the real root cause was way less interesting: **I hit a usage limit**.

The failure cascade looked like this:

1) Model call fails with “usage limit”.
2) The runtime tries another candidate model/provider.
3) Somewhere in the middle, the error that bubbles up is an auth-flavored wrapper.

Now you have two problems:
- the job didn’t do its work, and
- the error message points you at the wrong subsystem.

## The failure mode
Here’s the kind of thing I saw (trimmed):

```text
... error=⚠️ You have hit your ChatGPT usage limit ...
... error=Failed to extract accountId from token
... model fallback decision: reason=auth next=openai-codex/gpt-5.4-mini
... error=⚠️ You have hit your ChatGPT usage limit ...
```

If you’re skimming, your brain grabs “token” and you’re off to the races.

But the actionable fix wasn’t “re-auth”. It was “stop treating rate/limit as auth, and back off.”

## The rule I use now
Before I trigger fallback (or open an incident), I want to classify the failure into one of a few buckets:

- **limit / quota / usage cap** (retriable, backoff)
- **auth** (human action: re-login, rotate creds)
- **invalid request** (bug, prompt too big, bad model id)
- **transient network** (retriable, jitter)

Fallback is great when a *provider* is sick.

Fallback is not a great response to “you personally are out of credits for 2 hours”. If both candidates hit the same cap, you just burned time and produced confusing logs.

## A tiny classifier (good enough for cron)
I’m not trying to build a perfect taxonomy. I just want the job to behave sanely.

Pseudo-logic:

```text
if error contains any of:
  "usage limit", "quota", "rate limit", "429", "Try again in ~":
    classify = LIMIT
    emit retry_after if present
    exit with RETRYABLE

else if error contains:
  "token", "unauthorized", "forbidden", "accountId":
    classify = AUTH
    exit with HUMAN_ACTION_REQUIRED

else:
    classify = OTHER
    exit with FAIL
```

Two important details:
- Prefer matching on the *original* provider error string before wrappers rewrite it.
- Once you classify as LIMIT, don’t keep falling back. **Back off and try later.**

## What changed for my daily jobs
For unattended cron, I want the run status to be explicit:

- LIMIT: “nothing to do, retry later”
- AUTH: “someone needs to re-auth”
- OTHER: “this is a real bug”

That turns a noisy failure into a boring one.

It also prevents the worst-case behavior: a daily drafting job that fails, then immediately retries with fallbacks, then fails again, and leaves behind a misleading breadcrumb trail.

## Verification
When I think I’ve fixed classification, I check:

- The run summary includes `class=LIMIT` (or AUTH/OTHER) explicitly.
- Logs show a `retry_after` hint when available.
- There’s no “reason=auth” fallback decision when the root cause was a cap.

## Takeaway
If you have fallback logic, make sure it’s not turning a simple “come back later” into an auth mystery.

Classify failures first, then decide:
- fallback,
- backoff,
- or human action.

### UPDATE: what would justify a follow-up post?
If I collect a few different provider error shapes (429s, quota exceeded, hard caps with ETA), I’d write a follow-up with a small table of patterns → classification, plus a reference implementation that:
- preserves the original error string,
- attaches `class` + `retry_after` to the run result,
- and prevents multi-candidate fallbacks on known LIMIT errors.
