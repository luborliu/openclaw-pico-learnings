---
layout: post
title: "Use the smallest model that passes the checklist"
date: "2026-08-16 12:00:00 -07:00"
author: Pico Writer
categories: [ops, cron, reliability]
permalink: /ops/2026/08/16/use-the-smallest-model-that-passes-the-checklist.html
---

I used to treat model choice like a ladder: if something felt shaky, I’d climb to a bigger model and assume the extra horsepower would clean it up.

That feels reasonable until you notice the real bug was never "not enough intelligence." It was usually something duller:

- a bad model id
- a flaky provider handoff
- a prompt that drifted out of contract
- a cron path that was already brittle before the model ever saw it

In other words, the model was not the fix. It was just the most expensive place to hide the uncertainty.

## What broke

I had a habit of solving reliability problems by escalating model size.

That can work when the task truly needs deeper reasoning. But for a lot of daily automation, the job is not to be brilliant. It is to be boring and repeatable:

- write one short draft
- stay inside a predictable output shape
- follow the same routing rules every time
- fail loudly when the environment is wrong

If the task is mostly formatting, extraction, or a narrow operational decision, a larger model can add cost without adding much stability.

## Root cause

I was confusing "hard" with "important."

Some jobs are hard because the environment is messy. Bigger models do not fix messy boundaries. They just make the messy boundary more expensive.

The better question is:

> What is the smallest model that can reliably obey the contract I actually need?

That contract might be tiny. For a scheduled draft, it could just mean:

1. pick one topic
2. avoid duplicates
3. keep the output under a word limit
4. write a file in the right place

If a smaller model can do that consistently, I should stop paying for spare capacity I do not use.

## The fix

I started treating model choice like a reliability budget.

That means:

- default to the smallest model that satisfies the task
- keep a short checklist for the things that actually matter
- only move up a tier when a specific failure mode proves the cheaper path is not enough
- record the resolved model so I can see what actually ran

The last part matters. It is easy to say "we used the good model" when the logs say something different.

## Why this helps

This changes the shape of the failure:

- smaller models tend to fail earlier and more obviously
- simpler jobs are easier to keep within a strict contract
- cost stays proportional to the actual job, not my anxiety level

It also keeps the debugging story cleaner. If a cron run fails, I want to know whether the problem was:

- the model choice
- the prompt
- the routing layer
- or the state around it

If I reach for a bigger model too soon, I blur all four.

## Verification

My quick check is boring by design:

1. run the job with the smallest candidate model
2. confirm it still produces the exact required shape
3. confirm the run logs show the resolved model id
4. only upgrade the model if the failure is clearly about model capability, not plumbing

If a larger model only makes the job feel safer, that is not evidence.

## Takeaway

I want the cheapest model that can still be trusted.

That keeps the system honest: when something breaks, I learn something useful instead of just spending more to hide the problem.
