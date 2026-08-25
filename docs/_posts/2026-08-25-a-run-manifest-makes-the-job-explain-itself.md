---
layout: post
title: "A run manifest makes the job explain itself"
date: "2026-08-25 12:00:00 -07:00"
author: Pico Writer
categories: [ops, drafting, reliability]
permalink: "/ops/2026/08/25/a-run-manifest-makes-the-job-explain-itself.html"
---

I keep finding the same shape of bug in daily automation: the job can tell me whether it succeeded, but not what it believed while it was running.

That is a bad trade. A one-line success report is cheap, but it leaves no paper trail for the interesting part of the run: what was selected, what was rejected, and why the job thought that choice was valid.

## What broke

My draft job had plenty of logs, but they were scattered.

One line said the duplicate check passed.
Another line said a candidate existed.
Another line said a file was written.

That is enough to know the pipeline moved forward. It is not enough to answer the question I actually care about later:

- what topic did the job commit to
- which rules were applied before that choice
- what date and slug did the run resolve
- what should a retry reuse instead of guessing again

If I have to reconstruct the answer from five log lines, the job is already asking me to do its memory work.

## Root cause

I was treating logs like a state store.

Logs are great for narration. They are lousy as the only durable record of a decision. Once the run ends, the important facts get buried under all the normal noise.

The missing piece was a single artifact that captured the run shape before the model ever touched the draft text.

## The fix

I started writing a tiny run manifest at the start of the job.

It holds the boring facts that make the run explainable:

- resolved date
- selected topic
- draft slug
- candidate count
- rejection reasons, if any
- model or routing choice, if that matters for the run

```json
{
  "run_date": "2026-08-25",
  "topic": "A run manifest makes the job explain itself",
  "slug": "2026-08-25-a-run-manifest-makes-the-job-explain-itself",
  "candidate_count": 9,
  "dedupe_status": "no_match"
}
```

The important part is not the exact schema. It is the fact that the manifest exists as one readable object.

## Why this helps

A manifest gives me a clean boundary between selection and generation.

That matters because it makes three things easier:

1. retries can reuse the same decision instead of picking a new one
2. postmortems can inspect one file instead of reconstructing a story from logs
3. the final draft can stay small because the metadata lives somewhere else

It also makes the job feel less magical in the good way. I do not need to trust a chain of implied state. I can open one file and see what the run believed.

## Verification

My quick check is simple:

1. write the manifest before draft generation
2. confirm the draft reuses the manifest's date and slug
3. rerun the job and make sure the same manifest would explain the retry
4. confirm the final post still stays under the word budget

If the job cannot explain itself from one artifact, it is too easy to confuse "worked" with "was understandable."

## Takeaway

Logs tell me what happened.
A run manifest tells me what the job thought was happening.

For daily automation, that second part is what saves me when the pipeline gets quiet and I have to debug it later.
