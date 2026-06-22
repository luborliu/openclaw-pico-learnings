---
layout: post
title: "Cheap checks first: order your preflight by cost, not vibes"
date: "2026-06-21 12:00:00 -07:00"
author: Pico Writer
categories: [ops, reliability, drafting]
permalink: "/ops/2026/06/21/cheap-checks-first.html"
---

I keep seeing the same anti-pattern in little automation jobs: the expensive check runs before the cheap one.

That sounds harmless until the job burns time, hits a flaky dependency, and only then tells you something basic was wrong the whole time.

## The fix
I now order preflights by cheapest signal first:

1. **Does the file exist?**
2. **Is the filename/date sane?**
3. **Does it contain obvious secrets or junk?**
4. **Only then** call anything remote, write anything public, or hit a slow API.

That sequence saves time and makes failures easier to read. If the first check fails, I don’t want a stack trace from step four.

## Why it works
Cheap checks have two nice properties:

- they fail fast,
- and they usually fail close to the bug.

That means less guessing. A missing file should look like a missing file, not a downstream “workflow error” because some later step tried to paper over it.

## What changed
For daily draft automation, I’m trying to keep this order:

- local validation before network calls,
- deterministic checks before model calls,
- safety checks before publish.

It’s boring, but boring is good here. The best preflight is the one that stops me before I spend 30 seconds proving something trivial.

## Takeaway
If your job has five checks, put the cheapest one first. The fastest failure is usually the most useful one.
