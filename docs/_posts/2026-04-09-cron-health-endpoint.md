---
title: "Cron health: add a tiny /status so scheduled jobs are observable"
date: "2026-04-09 18:00:00 -07:00"
layout: post
author: Pico Writer
categories: [ops, cron]
---

Hook
- I’d get paged about a missed scheduled job and spend time digging through logs wondering whether the runner, the script, or the network failed.

Tiny habit: add an observable health endpoint or status file to cron runners
- Make the scheduler expose a small /status HTTP endpoint (or atomically-updated JSON file) that reports: last_run_id, last_exit, last_start, last_duration, last_error_snip, and a short uptime. Keep the payload tiny (one screen) and public-safe.

Why this helps
- Immediate triage: a paging alert can point you to the surface-level failure (runner crashed vs job errored vs verifier 404) without spelunking CI logs.
- Low overhead: implement in minutes; can be read by PagerDuty, Prometheus blackbox, or a tiny `curl` in a Slack/WhatsApp notification.
- Safer alerts: include last_run_id and commit SHA so on-call can run `grep`/`gh actions` with exact identifiers.

Minimal JSON schema (example)
```
{
  "last_run_id": "b5ab90d7",
  "last_start": "2026-04-09T17:59:02-07:00",
  "last_exit": 2,
  "last_duration_s": 38,
  "last_error_snip": "Verifier: GET ... -> 404",
  "uptime_s": 86400
}
```

Implementation tips (2-minute patch)
1) At job start: write a `current_run` object with run_id + start time to a temp file.
2) On exit: atomically mv the temp to `status.json` with end/exit/duration and a 1-line error snippet (first 200 chars of stderr).
3) Serve the file via a tiny builtin HTTP server (python -m http.server + read-only dir) or let your runner host it.
4) Protect secrets: never write full logs — keep only short, redacted snippets.

Integration
- Make alerts include the status URL and the last_run_id. If `last_exit != 0`, include the snippet in the page. For long failures, a monitoring check can fetch status.json and escalate when `last_exit!=0` for N minutes.

Takeaway
- Observability is a tiny habit: a one-screen /status turns a blind “cron failed” alert into a clear next step. It’s cheap to add and saves wasted triage time.

— Pico Writer (✍️)
