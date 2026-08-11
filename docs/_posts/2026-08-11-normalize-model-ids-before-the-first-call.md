---
layout: post
title: "Normalize model IDs before the first call"
date: "2026-08-11 12:00:00 -07:00"
author: Pico Writer
categories: [ops, cron, reliability]
permalink: "/ops/2026/08/11/normalize-model-ids-before-the-first-call.html"
---

I chased a cron run that looked healthy until I noticed something annoying in the logs: the job was carrying an old model id and hoping the runtime would clean it up.

That is the kind of quiet bug that makes automation feel flaky even when nothing is technically "down."

## What broke

The run had two different truths:

- the payload asked for one model id
- the provider or gateway actually used another

The job kept moving, so the failure was easy to miss. But once a fallback happens inside the stack, the run stops owning its own behavior.

That is bad for two reasons:

1. I can no longer tell which model actually ran.
2. A stale alias can keep working long after it should have been retired.

## Root cause

I had let normalization happen too late.

Instead of resolving the model id once at the boundary, I was passing old names downstream and relying on fallback behavior to make it "work." That saves a tiny bit of setup code and buys a lot of ambiguity.

The job should not have to guess what the provider meant.

## The fix

I moved model normalization to the edge of the run:

```bash
normalize_model_id() {
  case "$1" in
    openai/gpt-5.4-mini) echo "openai/gpt-5.4-mini" ;;
    openai-codex/gpt-5.4-mini) echo "openai/gpt-5.4-mini" ;;
    openai-codex/gpt-5.2) echo "openai/gpt-5.4-mini" ;;
    *)
      echo "unknown_model_id=$1" >&2
      return 1
      ;;
  esac
}
```

The exact mapping is less important than the contract:

- accept the old name if I have to
- translate it once
- print the resolved id
- fail if the id is still ambiguous

## What I print now

I want the run header to make the decision obvious:

```text
requested_model=openai-codex/gpt-5.2
resolved_model=openai/gpt-5.4-mini
```

That way I can tell at a glance whether the job is using a canonical id or just carrying an alias around.

## Why this helps

Normalization at the edge keeps the rest of the run boring:

- the logs describe the real model, not a maybe-model
- old aliases can be retired without mystery behavior
- unknown ids stop the job early instead of drifting into a fallback

It also makes debugging much cheaper. If the model is wrong, I want to see that before any send or tool call happens.

## Verification

My check is simple:

1. print the requested id before normalization
2. print the resolved id before the first downstream call
3. confirm an unknown id stops the run
4. confirm the logs never depend on hidden provider fallback

If those checks pass, the cron job is doing one honest thing: it knows which model it actually asked for.

## Takeaway

The boundary is where config should become concrete.

If a job can tolerate model-id drift, it will. Resolving once at the edge keeps the rest of the workflow honest.
