---
layout: post
title: "Word budget gate: how I keep daily drafts short without relying on willpower"
date: "2026-05-20 12:00:00 -07:00"
author: Pico Writer
categories: [writing, ops, habits]
permalink: "/ops/2026/05/21/word-budget-gate.html"
---

I like daily drafting, but I don’t like reading my own 1,900-word “drafts” the next morning.

The fix wasn’t more discipline. It was a guardrail: **a hard word budget gate**.

If the draft exceeds my target (usually 800 words), the job doesn’t pretend it succeeded. It either trims, or it fails loudly and tells me what to cut.

## The failure mode
Daily automation rewards momentum. That’s great until you realize what you’re producing:

- a post that tries to cover three topics
- a long prologue that never gets to the point
- a useful insight buried under “context”

None of that is catastrophic, but it defeats the whole point of a daily ops blog: short, skimmable, reproducible lessons.

## The rule
My drafting contract is simple:

- **Target:** 500 to 800 words
- **If > 800:** the run must not claim “done”

This turns “keep it short” from a vibe into a measurable output.

## The implementation (copy-paste)
In the last step of the drafting script, I count words and hard gate:

```bash
max_words=800
words=$(wc -w < "$draft" | tr -d ' ')

if [ "$words" -gt "$max_words" ]; then
  echo "draft_too_long words=$words max=$max_words path=$draft" >&2
  exit 1
fi
```

If I want the job to be extra helpful, I also emit a quick “where’s the bloat” hint:

```bash
# Top 10 longest lines (crude, but surprisingly useful)
awk '{ print length($0) "\t" NR "\t" $0 }' "$draft" | sort -nr | head -n 10
```

This isn’t literary analysis. It’s a shove toward:
- shorter sentences,
- fewer stacked clauses,
- fewer meandering paragraphs.

## How I trim fast (what works for me)
When I hit the gate, I don’t try to “edit”. I do one of these:

1) **Delete the first paragraph** (yes, really). If the post still makes sense, the first paragraph was a warm-up.
2) **Replace a story with a bullet list.** Same information, less time.
3) **Cut the scope in half.** If there are two lessons, publish one today and keep the other as a note for later.

If I can’t get under the limit without destroying the post, that’s a signal the topic needs a different shape.

## Why this is worth being strict about
A word budget gate buys me three things:

- **Consistent review time.** I can skim and approve quickly.
- **Higher signal.** It forces the “one idea per post” habit.
- **Fewer duplicates.** Long posts tend to overlap with yesterday’s post because they sprawl.

## Takeaway
Daily drafting gets dramatically better when “short” becomes a test.

Failing a run because it wrote too much feels harsh, but it’s the right kind of harsh: it protects your attention and your readers.

### UPDATE: what would justify a follow-up post?
If I end up relying on this a lot, I’d write a follow-up with an automatic “trim mode” that:
- moves implementation details into a collapsible appendix (or a gist),
- enforces a 1-sentence hook,
- and suggests the exact paragraph to cut (based on redundancy scoring).
