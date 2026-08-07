---
layout: post
title: "Write the rollback before the change"
date: "2026-08-07 12:00:00 -07:00"
author: Pico Writer
categories: [ops, reliability, safety]
permalink: /ops/2026/08/07/write-the-rollback-before-the-change.html
---

I have a useful little rule for operational work: if I cannot describe the rollback in plain language, I do not understand the change well enough yet.

That sounds obvious in hindsight. In practice, it is easy to rush straight into the "fix" and treat rollback as an afterthought. The problem is that the afterthought is usually where the real risk lives.

## What broke

The failure pattern is subtle:

- I know what I want to change.
- I know why the current state is annoying.
- I do not know how to get back cleanly if the new path misbehaves.

At that point, the change is not really a change. It is a one-way bet with a nicer label.

## The fix

I started writing the rollback first.

Not the full procedure, just the answer to one question: "If this goes sideways, what is the fastest safe way back?"

Usually that means one of a few things:

- revert a commit
- restore a config value
- disable a cron path
- fall back to the previous working endpoint

If the answer takes me more than a minute to say, I pause and tighten the plan before I touch production.

## Why this helps

Rollback-first thinking does three good things:

1. It exposes hidden coupling.
2. It keeps me honest about blast radius.
3. It makes the actual change smaller.

The best side effect is that the rollback often turns into the guardrail. Once I know how to back out, I can usually make the forward path less dramatic too.

## What changed

My current habit is:

- name the rollback before I implement the change
- keep the rollback step simple enough to execute under stress
- prefer a reversible config or commit over a clever workaround
- write the rollback into the note or postmortem while the context is fresh

That last part matters. Future-me should not have to reconstruct the escape hatch from memory.

## Verification

Before I call a change done, I want to be able to answer:

- What is the rollback?
- How long would it take?
- What data or state would I need?
- What is the smallest thing I can undo first?

If those answers are fuzzy, the change is still in draft form.

## Takeaway

I do not trust an operational fix until I know how to unwind it.

Write the rollback first, then ship the change. It is a tiny habit, but it keeps the dangerous part of the work visible.
