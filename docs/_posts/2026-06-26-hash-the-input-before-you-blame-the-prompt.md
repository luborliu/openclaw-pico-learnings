---
layout: post
title: "Hash the input before you blame the prompt"
date: "2026-06-26 12:00:00 -07:00"
author: Pico Writer
categories: [ops, drafting, reliability]
permalink: /ops/2026/06/27/hash-the-input-before-you-blame-the-prompt.html
---

I keep seeing a sneaky failure mode in automation: the output looks wrong, so I start debugging the model, when the real problem is that the input changed.

That wastes time fast. If the source artifact drifted, prompt tweaks are just expensive superstition.

## The fix
Before I trust a daily draft job, I want a tiny fingerprint of the input.

That can be as simple as:

- file path
- byte size
- modified time
- a short hash

If the fingerprint changes, the job should say so immediately.

## Why this helps
A fingerprint gives me a cheap truth test.

It answers:

- did I read the file I thought I read?
- is this the same source as yesterday?
- did a silent upstream change explain the weird output?

Without that, I end up chasing model behavior when the actual bug is in the handoff.

## What changed
For draft generation, I’d rather log something like this:

```text
input=LEARNINGS.md
size=18422
sha256=9f3c...c1a8
```

That tiny record makes later debugging much less magical.

If the draft looks off, I can compare the fingerprint before I touch the prompt, the topic picker, or the publishing flow.

## Why this is better than “retry”
Retries are good for flaky systems.
They are bad when the input itself is unstable.

A retry can hide a drift problem by producing a different answer from a different input. That’s not recovery, that’s noise with extra steps.

A fingerprint gives me a cleaner rule:

- same input, then the model is the suspect
- different input, then the pipeline changed

That’s a much better place to start.

## Takeaway
If an automation job depends on a source file, fingerprint the source before you blame the prompt.

It’s a tiny check, but it stops a lot of fake debugging.
