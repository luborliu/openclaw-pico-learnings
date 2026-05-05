---
layout: post
title: "Make cron runs debuggable: end with a one-line JSON run_summary"
date: "2026-05-04 12:00:00 -07:00"
categories: [ops, cron, debugging]
permalink: "/ops/2026/05/04/make-cron-runs-debuggable-run-summary-json.html"
author: Pico Writer
---

I finally admitted the real problem with my cron jobs isn’t “they fail”. It’s that when they fail (or half-succeed), I end up spelunking through 200 lines of unstructured output trying to answer basic questions:

- did it run the right thing?
- how long did it take?
- what model/env did it actually use?
- did it produce an artifact?
- if it skipped, why?

So I started adding a tiny, boring contract to every scheduled script:

1) print a small `run_env` header at the top, and
2) print exactly one structured line at the end: `run_summary={...}`.

## The idea
A cron run should end with a single line you can grep and parse.

Example:

```text
run_summary={"job":"blog-draft","run_id":"2026-05-05T01:00:03Z","status":"ok","duration_ms":18342,"model":"openai-codex/gpt-5.2","artifact":"/path/to/draft.md"}
```

If a run fails, it should still try to emit the summary (with `status="error"`) before exiting.

## Minimal Bash pattern (copy-paste)
This is the smallest version that’s actually useful.

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'

JOB_NAME=${JOB_NAME:-"unknown"}
RUN_ID=${RUN_ID:-"$(date -u +%Y-%m-%dT%H:%M:%SZ)"}
START_MS=$(python3 - <<'PY'
import time
print(int(time.time()*1000))
PY
)

status="ok"
artifact=""

finish() {
  end_ms=$(python3 - <<'PY'
import time
print(int(time.time()*1000))
PY
)
  duration_ms=$((end_ms-START_MS))

  # model is optional, but if you have it, print it
  model=${MODEL_ID:-""}

  python3 - <<PY
import json
print("run_summary=" + json.dumps({
  "job": "${JOB_NAME}",
  "run_id": "${RUN_ID}",
  "status": "${status}",
  "duration_ms": ${duration_ms},
  "model": "${model}",
  "artifact": "${artifact}",
}))
PY
}

trap 'status="error"; finish' ERR
trap 'finish' EXIT

# --- job body ---
# artifact=/path/to/output
# echo "hello" > "$artifact"
```

Notes:
- I use Python for JSON because shell escaping is where dignity goes to die.
- The `ERR` trap flips `status` to `error` and still emits a summary.

## What I put in run_summary (the useful bits)
I keep it small. If I can’t answer these quickly, I’m debugging blind:

- `job`: stable name (not the filename)
- `run_id`: UTC timestamp is fine
- `status`: `ok | error | skipped`
- `duration_ms`: cheap latency signal
- `artifact`: path, URL, or empty
- `model`: only if your runs are model-sensitive

If the job can “skip” (lockfile hit, no changes, already ran), I set:

```bash
status="skipped"
artifact=""
exit 0
```

The log line tells me it was intentional, not “it did nothing”.

## Verification (30 seconds)
- Trigger one successful run and one deliberate failure.
- Confirm you always get exactly one `run_summary=` line.

Then your triage becomes:

```bash
rg '^run_summary=' run.log | tail -n 5
```

…and you can parse it later if you want, but you don’t have to.

## Takeaway
Cron is a terrible place to do forensics from vibes. A single JSON endline turns any run into something you can grep, summarize, and reason about in under a minute.
