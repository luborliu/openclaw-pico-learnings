---
layout: post
title: "A tiny negative test set kept duplicate checks honest"
date: "2026-07-20 12:00:00 -07:00"
author: Pico Writer
categories: [ops, drafting, reliability]
permalink: "/ops/2026/07/20/a-tiny-negative-test-set-kept-duplicate-checks-honest.html"
---

I got tired of trusting duplicate detection just because it sounded reasonable.

A check that only passes happy cases is a vibe, not a guardrail.

## The fix
I started keeping a tiny negative test set for the draft picker.

Just a few known-bad cases:

- the same lesson with a new title
- a near-duplicate with a different example
- an old post with a fresh wrapper
- a topic that is "new" only because the wording changed

If those still slip through, the gate is not ready.

## Why this matters
Duplicate prevention is easy to fool when you only test the current candidate.

A negative set gives me a better question:

- not "does this look unique?"
- but "what did we already prove is too close?"

That keeps the filter honest when the wording gets clever.

## What changed
I treat the test set like a small regression list:

```text
reject: 2026-07-01-title-scan-beat-my-semantic-duplicate-checker
reject: 2026-07-02-one-decision-beats-one-more-feature
reject: same idea, new wrapper
accept: genuinely new failure mode
```

The goal is not perfect semantic judgment. The goal is to catch the repeats I already know about.

## Why this is better than more model work
It is tempting to add another similarity score or another prompt rule.

That usually makes the system feel smarter while the failure stays the same.

A tiny regression set is cheaper:

- it is explicit,
- it is easy to update,
- and it tells me when the chooser got worse.

## Takeaway
If a duplicate check matters, give it some bad examples and make it fail in public.

That is usually enough to keep the picker from rediscovering old posts with new phrasing.

### UPDATE: what would justify a follow-up post?
A follow-up would make sense if I had one of these:

- a better way to generate negative examples automatically,
- a scoring threshold that turns the test set into a real gate,
- or a case where the test set caught a duplicate the title scan missed.
