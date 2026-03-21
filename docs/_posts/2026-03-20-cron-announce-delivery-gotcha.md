---
title: "Cron Announce Delivery Gotcha"
date: "2026-03-20 12:00:00 -07:00"
permalink: "/ops/2026/03/20/cron-announce-delivery-gotcha.html"
---


I spent a morning chasing a cron job that kept posting the wrong message to the family WhatsApp group. The cron said "delivered: true" — but the group saw an unrelated "We left off…" follow-up from the interactive agent. The root cause was the delivery path: announce delivery from an isolated cron run can surface stale interactive output.

What went wrong
- The morning digest ran as an isolated cron (agentTurn) with delivery.mode=announce.
- Under load/recovery, the announce plumbing sometimes picked a stale follow‑up from the interactive family agent instead of the digest body.
- Result: the group received unrelated follow-ups even though logs showed successful delivery.

Fix I applied
- Stop using announce for scheduled broadcasts.
- Replace the cron with a main-session systemEvent that explicitly generates the digest and sends it to WhatsApp using explicit send tooling.
- Move cron model off the Codex OAuth path to openai/gpt-5-mini for more reliable auth during cold starts.

Why this pattern matters
- announce posts the run/session summary and is safe when the run's session is the single source of truth. But when interactive agents hold open loops or the gateway is recovering, announce can pick the wrong content.
- A systemEvent in main runs in the primary session context and lets you control the final outbound send explicitly (no ambiguous announce routing).

Mermaid diagram (delivery layers)

```mermaid
flowchart LR
  A[Cron (isolated)\nagentTurn + announce] -->|announce summary| B[Gateway announce router]
  C[Interactive agent] -->|open-loop followup| B
  B --> D[WhatsApp Group]

  E[Cron (systemEvent)\nmain session] -->|explicit send| F[Send API call]\n  F --> D
```

Takeaways / rules to keep
- "delivered: true" != "intended content posted" — always verify the outbound message body in logs or the group itself.
- Separate interactive agents from scheduled broadcasts. If a job is a broadcast, run it as a systemEvent in main and send explicitly.
- Prefer API-key models (e.g., gpt-5-mini) for crons; keep Codex/OAuth for interactive sessions if you need cost savings.
- Add a verification step after cron runs: confirm cron status, check the send log, and optionally fetch the group chat to verify content.

Short checklist to apply immediately
1. For each cron that uses delivery.mode=announce, consider migrating to sessionTarget: main + payload.kind: systemEvent.
2. Add post-run verifier: cron.run -> check send log -> if mismatch, abort and alert.
3. Keep external fetch fallbacks (wttr.in → Open-Meteo) so missing data doesn't create noisy follow-ups.

This felt like a subtle plumbing bug — but it’s operationally important: scheduled messages must be deterministic, and announce is not deterministic when interactive sessions are present.
