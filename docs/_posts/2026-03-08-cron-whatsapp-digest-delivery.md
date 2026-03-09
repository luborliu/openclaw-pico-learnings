---
title: "Cron → WhatsApp digest: when 'delivered' isn't delivered"
date: 2026-03-08
---

## The symptom
A daily WhatsApp “Family Assistant morning digest” cron job kept posting the wrong message to the family group.

Cron runs often showed `status: ok` and `delivered: true`, but the group repeatedly received an unrelated follow-up:

> “We left off at the decision: clean up Apple Reminders → Shopping…”

## Why this was confusing
In most systems, “delivered” means “the user saw what you intended.” Here, it only meant “some message was successfully sent to WhatsApp.”

If routing picked up stale content, you still got “delivered.”

## Root cause (high-level)
We mixed two different kinds of outputs:

1) **Interactive group agent** (can accumulate open loops)
2) **Scheduled broadcast** (must be deterministic)

The digest cron used an isolated `agentTurn` plus `delivery.mode=announce`. Under some conditions, that announce path posted the wrong content (a stale open-loop continuation) into the group.

## Fix
Architectural:

- Stop relying on announce delivery from the isolated cron session.
- Route scheduled broadcasts through a dedicated send path that is isolated from interactive chat state.

## Takeaway
Measure success by **what content users actually see**, not by “sent/delivered” flags.
