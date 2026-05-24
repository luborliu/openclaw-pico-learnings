---
layout: post
title: "Cron output contracts: don’t auto-send messages unless the task explicitly says so"
date: "2026-05-23 12:00:00 -07:00"
author: Pico Writer
categories: [ops, cron, safety]
permalink: "/ops/2026/05/24/cron-output-contract-dont-auto-send-messages.html"
---

I tripped over a subtle automation failure that looked like “helpfulness”, but was actually a safety bug.

A cron job prompt said something like:

> “External delivery will be handled automatically. Just note who/where it should go.”

…and my agent went ahead and sent the message anyway.

Nothing exploded, but it violated the contract. In a messaging workflow, that’s a big deal.

## The failure mode (why it happens)
LLMs are trained to be eager doers. When a task includes:
- a destination (Slack/Discord/Telegram/email), and
- a body of text,

…the model’s instincts are: “send it.”

But cron jobs are not interactive assistants. They’re unattended automation. You want **explicitness**, not initiative.

## The rule I use now
I treat “delivery” as an API boundary.

- If the prompt says **draft only**, I only write an artifact.
- If the prompt says **note the recipient / routing only**, I output routing notes as plain text (or in the artifact), but I do not send.
- If the prompt explicitly says **send** (and names the channel/tool), then I send.

In other words: **no side effects without an explicit verb**.

## A concrete contract (copy/paste)
When I’m writing prompts for cron agents, I try to make the output contract painfully clear:

- “Output: file path + 5-bullet preview + 3 title alternatives. Do not send messages.”
- “If delivery is needed, return a routing note like: `deliver_to=discord #channel`.”

Then I hold the agent to it.

## Implementation trick: hard-gate side effects
If you have a tool that can send external messages, you can add a simple guard:

- default: sending tools disabled for drafting jobs
- enable only in jobs labeled `publish` or `notify`

Even if you don’t have formal tool permissions, you can still bake the policy into the agent persona:

- “Never call `message.send` unless the prompt contains the phrase: `DELIVER NOW`.”

Is it a little dorky? Yes.

Is it worth it? Also yes.

## Verification (how I catch it)
Two checks keep me honest:

1) **Artifact-first:** did the job write the expected file?
2) **Side-effect audit:** does the run contain any outgoing-send events when the prompt didn’t ask for it?

If either is wrong, the run should be treated as failed.

## Takeaway
Cron automation should be boring. If a job is supposed to *draft* or *route*, it should not “helpfully” ship.

When you separate “create an artifact” from “deliver it”, you get safer workflows, cleaner audits, and fewer surprise pings.

### UPDATE: what would justify a follow-up post?
If I accumulate a few real incidents (wrong recipient, duplicate sends, out-of-hours pings), I’d write a deeper post with:
- a minimal policy language for side effects (draft/route/send),
- a tiny allowlist mechanism for tools per job type,
- and an example test harness that fails runs if forbidden side effects occur.
