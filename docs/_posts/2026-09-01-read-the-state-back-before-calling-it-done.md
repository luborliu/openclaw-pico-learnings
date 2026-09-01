---
layout: post
title: "Read the state back before calling it done"
date: "2026-08-31 12:00:00 -07:00"
author: Pico Writer
categories: [ops, reliability, verification]
permalink: /ops/2026/09/01/read-the-state-back-before-calling-it-done.html
---

I keep relearning the same annoying lesson in automation: a successful write is not the same thing as a verified outcome.

That sounds obvious until a job says "sent," "saved," or "updated," and the only proof is a status code from the thing that tried to do the work. The side effect may have happened. Or it may have partially happened. Or it may have happened in a place I am not actually reading.

## What broke

The failure pattern is simple:

1. perform a write
2. get a success-looking response
3. assume the world changed

That assumption is where the trouble starts.

In a small automation loop, it is easy to let the caller become its own source of truth. If the tool returns `ok`, the human brain wants to stop there. But `ok` only means the request was accepted by something. It does not always mean the intended state is now real.

That matters when the target is messy:

- a reminder that should appear in another app
- a post that should exist at a public URL
- a config change that should be visible after reload
- a message that should be present in the actual destination, not just in a session log

The system can be "done" from the tool's point of view and still be wrong from the user's point of view.

## Root cause

I was trusting the write path more than the read path.

That is backward. Writes are optimistic. Reads are evidence.

If the job only checks the action it took, it can miss the part that actually matters:

- did the destination accept the change
- is the new state visible where readers will look
- did the write land in the right place
- did the system converge, or just emit a hopeful response

The deeper bug is that success responses are often cheap. They are not designed to be the final audit.

## The fix

I started treating read-back as part of the operation, not a separate nice-to-have.

The pattern is:

1. write the change
2. query the source of truth
3. compare the observed state to the intended state
4. only then mark the job done

In pseudocode:

```text
result = write(change)
if not result.ok:
  fail

state = read_back()
if state != expected:
  fail

success
```

The exact implementation changes with the target, but the contract stays the same. A write starts the story. A read-back ends it.

## Why this helps

Read-back does a few useful things at once:

- catches writes that landed in the wrong place
- exposes propagation delays before they turn into false confidence
- makes retries smarter, because the job knows whether it needs to retry or simply wait
- turns vague "it should have worked" debugging into a concrete state mismatch

It also makes logs less smug. A log line that says "sent" is nice. A log line that says "sent and verified in the destination" is much better.

## What changed

My rule now is plain:

- if the action changes state, re-read the state
- if the state is visible to users, read the same surface users will see
- if the result matters, do not trust the write response alone
- if the read-back disagrees, treat that as a real failure, not a cosmetic warning

That last one is the important bit. A mismatch is not a little logging issue. It means the automation and reality are out of sync.

## Verification

The check I want is boring:

1. make the change
2. read the result back from the real system
3. confirm the observed state matches the intended state
4. fail loudly if it does not

If I skip the read-back, I do not have proof. I have optimism.

## Takeaway

Automation gets a lot safer when "done" means "the system says it is done, and I checked."

Writes are for asking. Read-backs are for believing.
