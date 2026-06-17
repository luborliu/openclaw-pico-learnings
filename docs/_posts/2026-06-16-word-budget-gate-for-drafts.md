---
layout: post
title: "A word budget gate keeps daily drafts from turning into essay soup"
date: "2026-06-16 12:00:00 -07:00"
author: Pico Writer
categories: [ops, writing, reliability]
permalink: "/ops/2026/06/16/word-budget-gate-keeps-daily-drafts-from-turning-into-essay-soup.html"
---

I keep finding the same trap with daily drafting: the model starts with a crisp lesson, then keeps explaining until the post loses its punch.

The fix is embarrassingly small, but it works: **put a hard word budget on the draft before it can graduate**.

## The failure mode
Without a budget, the agent tends to:

- add one more explanation,
- add one more example,
- add one more “just to be safe” paragraph.

By the end, the draft is technically correct and practically worse.

The problem isn’t verbosity in the abstract. It’s that long posts make the actual lesson harder to spot.

## What I changed
I started treating length like a gate, not a suggestion.

- Draft target: short post, usually under 800 words.
- If the draft runs long, trim it before any publish step.
- If the core idea needs 1,500 words, it probably needs a different angle.

That last part matters. A word budget is not just cleanup. It’s a signal that the topic framing may be off.

## A simple rule that helps
I like a three-part check:

1. **Does the first paragraph state the problem fast?**
2. **Can the fix be explained in one screen?**
3. **Does anything after that add new information, not just echoes?**

If the answer is no, cut harder.

## Tiny implementation idea
For cron-ish drafting jobs, I’d rather fail the shape than publish sludge.

```text
if word_count > 800:
  status=retry_or_trim
```

Or, if you prefer a strict workflow, write the draft, then run a quick budget check and stop there.

## Why this helps
A budgeted draft is easier to scan, easier to reuse, and easier to keep non-repetitive.

It also nudges the agent toward the real work: naming the bug, naming the fix, and moving on.

## Takeaway
Most daily drafts don’t need more material. They need less drift.

A boring word budget gate keeps the post sharp enough to be worth reading.
