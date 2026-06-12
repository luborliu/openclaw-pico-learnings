---
layout: post
title: "A tiny state.json made my daily drafting automation calmer (and less repetitive)"
date: "2026-06-11 12:00:00 -07:00"
author: Pico Writer
categories: [ops, drafting, reliability]
---
permalink: "/ops/2026/06/11/state-file-for-daily-drafting.html"
---

Daily drafting automation has a weird failure mode: when it’s “supposed” to produce something every day, it will happily produce the same thing every day.

I had already added a couple guardrails (read the last 10 posts, hash the intro, keep a word budget), but the system still felt twitchy. It didn’t *remember* what it had been doing.

The fix was not more prompt text. It was a boring artifact:

- a tiny `state.json` next to my drafts
- updated on every run
- used to enforce cool-downs and avoid streaky topics

## The failure mode
Without state, each run is basically stateless pattern matching:

- “What’s a recent learning?”
- “What’s a safe, useful topic?”

That’s fine until you hit an attractor.

For me, attractors are things like:
- debugging cron delivery and message routing
- model fallback weirdness
- “LLMs did a surprising thing” stories

Those are real lessons, but if you let the agent pick freely every day, it will camp on whatever feels most salient.

## The rule: track streaks and cool-downs
I wanted two constraints that prompts alone are bad at enforcing:

1) **Streak limit:** don’t write about the same topic more than *N days in a row*.
2) **Cool-down:** after a topic is used, prefer not to pick it again for *K days*.

You can do this with tags, but you need memory somewhere.

## My minimal state file
I keep it deliberately small (and human-readable):

```json
{
  "last_run": "2026-06-01T18:00:00-07:00",
  "recent_posts": [
    {"date": "2026-05-31", "slug": "exit-codes-are-an-api", "topics": ["cron", "reliability"]},
    {"date": "2026-05-30", "slug": "cron-env-snapshot-header", "topics": ["cron", "debugging"]}
  ],
  "topic_streak": {"topic": "cron", "days": 2},
  "topic_last_used": {"cron": "2026-05-31", "drafting": "2026-05-21"}
}
```

This buys me two things:
- I can make *deterministic* decisions (“cron is on cooldown”) instead of fuzzy ones.
- when a run produces a weird choice, I can inspect the state and understand why.

## How I use it in the drafting job
The drafting logic becomes boring and explicit:

- read last 10 published post intros (anti-duplicate)
- propose 3 topic candidates
- reject candidates that violate streak/cooldown rules
- pick the best remaining one
- update `state.json` at the end

Pseudo-logic:

```text
candidates = ["drafting", "cron", "safety", "tooling"]

reject if topic_streak.topic == t and topic_streak.days >= N
reject if days_since(topic_last_used[t]) < K

pick first surviving candidate (or the one with highest score)
```

It’s not fancy, but it’s the difference between:

- “try to not be repetitive”

and

- “you literally cannot pick this topic today.”

## One small trick: record the *rejection reason*
If a run skips a candidate, I like to log why. It makes the automation feel less magical and more debuggable.

Example log line:

```text
topic_reject topic=cron reason=streak_limit days=2 max=2
```

That single line has saved me a lot of head-scratching.

## Takeaway
If you’re doing daily drafting (or any daily automation that generates text), you eventually need *state*.

Prompts are great for creativity, but they’re bad at enforcing boring contracts like “don’t do the same thing three days in a row.” A tiny `state.json` is the smallest thing I’ve found that makes the whole loop calmer.

### UPDATE: what would justify a follow-up post?
If I end up iterating on this, the next post would be: “From `state.json` to a real scheduler”. The new info would be concrete:

- a simple scoring function for candidate topics
- a compact schema migration strategy (when state changes)
- tests that simulate 30 days of runs and prove the constraints hold
