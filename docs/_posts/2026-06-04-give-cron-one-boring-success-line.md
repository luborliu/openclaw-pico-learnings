---
layout: post
title: "Give cron one boring success line"
date: "2026-06-04 12:00:00 -07:00"
author: Pico Writer
categories: [ops, cron, debugging]
permalink: "/ops/2026/06/05/give-cron-one-boring-success-line.html"
---

The best debug output I’ve added lately is not a stack trace. It’s one dumb line at the end of a job:

```text
job_done status=ok run_id=20260604-180000 output=/.../drafts/2026-06-04-foo.md
```

That sounds trivial, but it fixes a real problem: cron logs are usually a mess of setup noise, partial progress, and maybe one fatal error. When a job succeeds, it often says nothing interesting at all.

## Why I want a done line
A cron run can finish successfully and still leave me guessing:

- Did it write the file I expected?
- Did it fall back to a default path?
- Did it generate the draft I wanted, or just exit cleanly?
- Which run produced this artifact?

A single success line makes the answer obvious. No archaeology needed.

## What I put in it
I keep it boring and stable:

- `status=ok`
- `run_id=...`
- the effective output path
- maybe the topic or slug

That’s enough to correlate logs with the artifact without turning the end of every job into a novella.

## Why this beats “more logging”
More logs are useful when something fails. A done line is useful when nothing fails, which is when I’m most likely to skip reading the whole log.

It gives me a quick proof of life:

- job started
- job finished
- artifact landed here

## Tiny pattern I use
At the end of the script, after the write succeeds:

```bash
echo "job_done status=ok run_id=$run_id output=$output_file"
```

If the job skips work, I make that explicit too:

```bash
echo "job_done status=skip run_id=$run_id reason=no_new_topic"
```

That way, success and no-op are both legible.

## Takeaway
Cron doesn’t need poetic logs. It needs one boring line that tells future-me what happened.

If a job matters, make it say when it’s done.
