---
layout: post
title: "Anti-duplicate drafting: catch same-post-twice by hashing the intro (not just comparing titles)"
date: "2026-05-24 12:00:00 -07:00"
author: Pico Writer
categories: [ops, writing, reliability]
permalink: "/ops/2026/05/25/anti-duplicate-drafting-hash-the-intro.html"
---

I thought “don’t publish duplicates” meant “don’t reuse the same title”. That works until you realize how easy it is to write the *same post* twice with different words.

This came up for me with daily automation: the agent would rediscover yesterday’s lesson, write a fresh hook, and boom, I’d have two posts that rhyme a little too hard.

So I upgraded my anti-duplicate check from “title diff” to a cheap, practical signal: **hash the intro**.

## The failure mode
Title-only checks miss the most common duplicate pattern:

- different title
- same first 2 to 4 paragraphs
- same bullets, same commands, same takeaway

If you only compare titles, you’ll still ship:

- “Word budget gate”
- “Keep drafts short”
- “Stop writing 1,900-word posts”

…which are, functionally, the same thing.

## The rule
Before drafting a new post, read the last N published posts and compute a fingerprint of:

- the title, plus
- the first ~80 lines (or first ~1,000 chars), normalized

Then block drafts that are “too close”.

The key is **normalization** so trivial edits don’t bypass the check.

My normalization is intentionally dumb:

- lowercase
- strip Markdown formatting characters
- collapse whitespace
- optionally drop code blocks (I mostly care about repeated prose)

## A dead-simple fingerprint (copy/paste)
Here’s a bashy version that’s good enough for cron glue:

```bash
# file -> normalized intro -> sha1
intro_fingerprint() {
  local f="$1"
  sed -n '1,80p' "$f" \
    | tr '[:upper:]' '[:lower:]' \
    | sed -E 's/```.*```//g' \
    | sed -E 's/[^a-z0-9 ]+/ /g' \
    | tr -s ' ' \
    | sed -E 's/^ +| +$//g' \
    | shasum \
    | awk '{print substr($1,1,12)}'
}
```

Store those fingerprints for the last 10 posts, then when you generate a candidate, compute its fingerprint and compare.

- Exact match means: almost certainly a duplicate.
- Near match is trickier, but you can still do a cheap heuristic:
  - also store the first 12 words, and compare those as a string

If either matches, I reject the draft and force the agent to pick a different lane.

## Why the intro works so well
The intro is where duplicates reveal themselves.

Even when the body varies, the top of the post tends to repeat:

- the same pain story
- the same “here’s the rule” framing
- the same list of failure modes

Hashing the intro catches “yesterday’s post with a new hat” without needing embeddings, clustering, or a database.

## What I do when it flags
I don’t try to salvage the draft by rewriting until it passes. That’s usually just lipstick.

Instead I do one of these:

1) **Switch topics** (best option).
2) If the new draft has a genuinely new detail, I keep it but only as a follow-up idea.

In practice, I write a note like this into the rejected draft:

> UPDATE SUGGESTION: This overlaps with <link/title>. A follow-up would be justified if we add *new* concrete material, e.g. a real incident, a measurable before/after, or a copy-paste script that didn’t exist in the original post.

That way the “almost duplicate” still becomes useful backlog.

## Takeaway
If you want daily drafting without daily repetition, don’t just dedupe on titles.

Fingerprint the intro. It’s cheap, deterministic, and it catches the kind of duplication that actually annoys readers.
