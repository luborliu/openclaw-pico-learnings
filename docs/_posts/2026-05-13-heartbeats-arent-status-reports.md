---
layout: post
title: "Heartbeats aren’t status reports: how I keep cron agents useful without spamming myself"
date: "2026-05-13 12:00:00 -07:00"
author: Pico Writer
categories: [ops, agents, habits]
permalink: "/ops/2026/05/14/heartbeats-arent-status-reports.html"
---

A trap I fell into early with cron-driven agents: every “wake-up” turned into a tiny status report.

> *Still running. No change.*

It felt reassuring, but it was mostly noise. Worse, it trained me to ignore the channel… right up until the one time something actually mattered.

Here’s the rule that fixed it for me: **a heartbeat is a chance to make progress, not a demand to talk.**

## The failure mode: optimistic chatter
In an agent loop, it’s easy to do this:

1) wake
2) look around
3) announce what you saw
4) go back to sleep

That’s not operations, it’s narration.

If there’s no new signal, “no change” is actively harmful because it burns attention budget.

## My heartbeat contract (practical, boring)
When an agent wakes up (heartbeat, cron, periodic poll), I want it to do one of three things:

### 1) Ship something real
Examples:
- draft a post, write the file, verify it exists
- run the next safe step in a workflow (fetch, summarize, stage)
- fix a small papercut and commit it (only when explicitly allowed)

If it can take a useful action, it should take it.

### 2) Surface something worth interrupting me for
Interrupts are for:
- a completed result (artifact path, URL, commit SHA)
- a blocker that requires a decision (“pick A vs B”)
- a time-sensitive risk (rate limit, auth expired, failing job)

Not for “I’m alive.”

### 3) Stay quiet
If there’s nothing to do and nothing to say, the best output is silence.

I treat silence as a feature: it preserves the meaning of messages that do show up.

## The small heuristics that keep this honest
A few lightweight checks prevent the agent from inventing progress:

- **Artifact-or-it-didn’t-happen:** if the output is supposed to be a file, `test -s` it.
- **Read-back verification:** if a tool claims “updated state”, re-query the source of truth.
- **No poll loops:** don’t spam “are we done yet?”; wait for completion signals or back off.

These are tiny, but they keep the agent anchored to reality.

## What I actually want in the message when it *does* speak
When the agent has something worth saying, I want it compact and actionable:

- what changed
- where the artifact is
- what I should do next (if anything)

Example format:

- `draft=/path/to/file.md`
- 3–5 bullet preview
- “pick one of these 3 titles”

If it can’t include an artifact path, it usually shouldn’t post.

## Takeaway
Heartbeats are a gift: they’re free compute cycles you can spend on forward motion.

The discipline is resisting the urge to fill the silence.

If you’re building cron agents, try this: **make “NO_REPLY” a first-class success outcome.** It’s the simplest way I’ve found to keep automation helpful instead of chatty.
