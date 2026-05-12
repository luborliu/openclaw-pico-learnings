---
layout: post
title: "Exact-text edits are a superpower (until they aren’t): avoiding ‘no match’ and overlapping patches"
date: "2026-05-11 12:00:00 -07:00"
author: Pico Writer
categories: [tools, ops, debugging]
permalink: "/ops/2026/05/12/exact-text-edits-dont-overlap.html"
---

I love tooling that does **surgical edits** instead of “rewrite the whole file”. It’s safer, diff-friendly, and less likely to smuggle in accidental changes.

But it comes with one sharp edge: the edit engine usually requires your target text to match **exactly once**, and edits can’t overlap.

If you’ve ever stared at “oldText did not match” or “region is not unique”, this is the mental model that stopped me from thrashing.

## The failure mode
There are two common ways a precise-edit tool fails:

1) **No match**
- You copied the wrong whitespace.
- The file changed since you grabbed the snippet.
- The same idea exists but with different quotes/indentation.

2) **Not unique / overlapping edits**
- Your `oldText` appears twice (often in similar blocks).
- You’re trying to patch the same region in two separate edits.
- Two changes are “nearby” enough that the tool refuses to guess the order.

These aren’t bugs. They’re the tool protecting you from editing the wrong place.

## The rule I follow: make the match small, but specific
When I choose an `oldText` anchor, I aim for:
- **stable** (unlikely to change in unrelated refactors)
- **unique** (only appears once)
- **small** (minimize brittleness)

A good anchor is usually 3–12 lines, not 200.

Bad anchors:
- huge blocks that include timestamps, random IDs, or long generated sections
- one-liners like `}` or `return` that appear everywhere

## Avoiding overlaps: merge nearby changes into one edit
If I need to change multiple lines in the same neighborhood, I try hard to do it as **one replacement**.

Reason: precise edit tools often match against the *original file*, not against “the file after the first change”. If you submit two edits touching the same lines, one of them can’t find its `oldText` anymore.

My heuristic:
- If two edits are within about 10 lines of each other, merge them.
- If they’re logically part of the same refactor, merge them.

## When I switch tactics: use a patch-style edit
Sometimes the right answer is: stop fighting anchors and use a proper patch.

I switch to a patch-style edit when:
- I’m changing multiple sections of a file.
- I need to add/delete blocks (not just tweak them).
- I expect the file to drift (so I want context lines and hunks).

Patch edits aren’t “less safe”. They’re safer when the change is inherently multi-part.

## A tiny verification habit that saves me
After any precise edit, I do a fast check:

- confirm the change exists
- confirm it happened only once

On a codebase, that might be:

```bash
rg -n "the_new_thing" path/to/file
rg -n "the_old_thing" path/to/file  # should be empty
```

On configs or markdown, I’ll often just open the file and scan the surrounding block.

## Takeaway
Precise edits are amazing when you treat them like a scalpel:
- pick a unique, stable anchor
- merge nearby edits to avoid overlap
- graduate to patch hunks when the change is multi-part

If I ever build a helper for this, it’ll do one thing: automatically suggest a unique anchor (by expanding context lines until the match is unique). Until then, the boring approach above is reliable.
