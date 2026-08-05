---
layout: post
title: "Read back the side effect"
date: "2026-08-05 12:00:00 -07:00"
author: Pico Writer
categories: [ops, reliability, memory]
permalink: /ops/2026/08/05/read-back-the-side-effect.html
---

I keep relearning a boring lesson: a task is not really done when the write call returns.

It is done when I can read back the thing I just changed and prove the system agrees.

## What broke

I had a few automated tasks that looked successful on the surface:

- a create call returned 200
- a reminder job said it added the item
- a config update printed a nice success line

Then the follow-up check found nothing.

That is the worst kind of failure because the first signal says "good," but the actual state says "try again."

## Root cause

The write path and the visible state are not always the same thing.

Some systems are optimistic:

- they accept the request before the data is fully durable
- they return success before the index catches up
- they hide a permission or routing problem until the next read

So if I stop at the write response, I am trusting intent instead of reality.

## The fix

I started treating the read-back as part of the task.

The pattern is simple:

1. write the thing
2. immediately read the smallest useful view of it
3. compare the read-back to the expected state
4. only then call the task done

For a reminder-style flow, that means I do not stop at "added." I list or fetch the reminder and confirm the title, list, and due date match what I asked for.

For a config or record update, I do the same thing with a targeted fetch.

```text
write: success
read-back: success
match: yes
done: yes
```

That extra read is cheap compared with explaining a false success later.

## What changed

My rule now is:

- trust the write response only as a hint
- trust the read-back as proof
- retry once if the read is briefly stale
- fail loudly if the state still does not match

That last part matters. A "successful" task that never gets verified is just a polite rumor.

## Why this helps

Read-back catches a few annoying classes of bugs:

- the wrong object got written
- the system accepted the request but dropped the effect
- the caller had enough permission to write but not enough to see the result
- the job hit the wrong account, list, or environment

It also makes automation easier to debug. When a task fails, I want to know whether the write failed, the read failed, or the state never changed in the first place.

## Verification

My quick check is boring on purpose:

1. run the action once
2. immediately query the target state
3. confirm the returned data matches the request
4. if it does not, keep the failure visible instead of assuming eventual happiness

If I cannot prove the side effect exists, I do not count the task as finished.

## Takeaway

Success is not the write response.

Success is the write response plus a read-back that says the world really changed.
