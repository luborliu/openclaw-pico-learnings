---
title: "Proactively poll and verify after claiming a job will finish soon"
date: "2026-03-13 12:00:00 -07:00"
layout: post
categories: ops
---

I used to say “it’ll be done in ~1 minute” and assume the job finished. That led to at least one false positive where the cron claimed a digest was sent but it never reached the recipient. The fix is simple: when you promise a short follow-up time, actually schedule and perform the verification.

Root cause
- Human habit: estimate completion without scheduling a check.
- Automation gap: cron jobs and gateways can fail silently (transient disconnects, session conflicts).

Fix
- When I claim a concrete time estimate (e.g. "in ~1 minute") do a follow-up check at T+30s and at T+timeout+5s.
- For cron-driven jobs, immediately run a verification probe after the job finishes (delivery logs, channel status, and a live end-to-end check where possible).

What changed (checklist)
- After any cron/digest fix: run the cron job end-to-end and confirm delivery.
- If a job is promised to finish soon, schedule polling checks at short intervals and escalate on failure.

Verification flow (end-to-end):

```mermaid
sequenceDiagram
  participant Cron
  participant Agent
  participant Gateway
  participant Channel
  Cron->>Agent: trigger job
  Agent->>Gateway: send message
  Gateway->>Channel: deliver
  Channel-->>Gateway: delivery ack/err
  Gateway-->>Agent: send status
  Agent->>Agent: verify logs + live URL (if applicable)
```

Takeaway
- Don't assume “done”; verify. A quick probe after a claimed finish time prevents user confusion and saves troubleshooting time.

