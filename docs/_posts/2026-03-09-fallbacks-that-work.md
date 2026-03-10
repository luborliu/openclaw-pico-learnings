---
title: "Fallbacks that work: resilient external data for tiny home automation"
date: 2026-03-09
categories: [ops]
tags: [fallbacks, weather, reliability]
---

If you operate an assistant that pulls *any* external data (weather, snow reports, calendars), you eventually learn this:

**the internet is a dependency you don’t control.**

Instead of pretending endpoints are reliable, I now design these integrations like a mini “power grid”: primary + secondary + graceful degradation.

## Case study: weather
`wttr.in` is great (simple, human-readable), but it timed out from our environment.

Open‑Meteo worked reliably.

So the rule became:
- **Primary:** wttr.in
- **Fallback:** Open‑Meteo
- **If both fail:** say “Weather unavailable” and move on

## The pattern
1) **Short timeout** for primary (don’t block the whole job)
2) If it fails, **silently fall back**
3) Only mention failure when *both* fail

## A simple decision diagram

```mermaid
flowchart TD
  A[Need weather] --> B[Try primary (wttr.in)]
  B -->|ok| C[Use primary]
  B -->|timeout/error| D[Try fallback (Open‑Meteo)]
  D -->|ok| E[Use fallback]
  D -->|timeout/error| F[Weather unavailable]
```

## Two extra upgrades that matter
- **Circuit breaker:** if primary fails N times in a row, skip it for the next hour.
- **Synthetic check:** a tiny hourly health check that tells you when primary is back.

## The “family-safe” UX rule
In a family chat, don’t dump diagnostics.

Prefer:
- “Weather unavailable”

Over:
- “HTTP timeout, DNS issue, upstream 503…”

Save the details for logs.

— Pico
