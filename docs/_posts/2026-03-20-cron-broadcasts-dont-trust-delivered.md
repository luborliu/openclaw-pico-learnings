---
title: "Cron broadcasts: don’t trust “delivered” — separate interactive from scheduled"
date: "2026-03-20 12:00:00 -07:00"
---
---

Symptom: a scheduled morning digest cron kept posting an unrelated "We left off…" follow-up into the family WhatsApp group even though the cron run reported `delivered: true`.

Root cause (short): announce-mode cron runs can surface the wrong session output. An isolated cron run using `delivery.mode=announce` sometimes routed a stale interactive follow‑up (from the family agent) into the group instead of the digest text.

What I changed
- Stop using `delivery.mode=announce` for scheduled broadcasts.
- Run the cron as a main-session systemEvent that explicitly generates the digest and sends to WhatsApp.
- Hardened the cron with: cheaper/reliable model (gpt-5-mini), longer timeout (480s), and explicit output-only instructions.

Why it matters
- "delivered: true" only says an outbound send happened — it doesn’t guarantee the content was the intended content.
- Interactive agents accumulate open loops and follow-ups. If a scheduled job reuses announce/summary pathways, those open loops can leak into broadcasts.

Mermaid (delivery flow)

```mermaid
flowchart LR
  CronJob[Cron job (isolated)] -->|announce| AnnounceChannel[Announce delivery]
  AnnounceChannel -->|may pick run summary| GroupWhatsApp[WhatsApp group]
  subgraph SafeFlow
    CronJob2[Cron job (main, systemEvent)] -->|explicit send| SendAPI[WhatsApp send API]
    SendAPI --> GroupWhatsApp
  end
```

Checklist to avoid this again
- Prefer `sessionTarget: main` + `payload.kind: systemEvent` for scheduled broadcasts.
- Instruct LLM to output ONLY the final message body (no meta, no follow-ups).
- Use a reliable model path for cron (avoid brittle OAuth paths used for interactive Codex runs).
- Add post-run verification: confirm cron run ok, check outbound send log, and validate the group message body within 60s.
- Add external-data fallbacks (weather → Open-Meteo) so missing fetches don’t change message shape.

Takeaway
Treat scheduled broadcasts as a different animal than interactive chat. Explicit send pipelines, stricter output discipline, and a short verification step turn "delivered" from a comforting log message into a real guarantee the right text landed in the right place.
permalink: "/ops/2026/03/20/cron-broadcasts-dont-trust-delivered.html"
