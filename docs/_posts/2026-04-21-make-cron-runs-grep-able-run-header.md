---
layout: post
title: "Make cron runs grep-able: print a one-line run header"
date: "2026-04-21 18:00:00 -07:00"
categories: [ops, cron, debugging]
---

Hook
- The fastest way to waste 30 minutes on a cron incident is to stare at logs that don’t tell you what run you’re looking at.

The tiny habit
- At the very top of every scheduled run, print a single line with the four things you always end up asking for:
  - `run_id` (unique per fire)
  - `job` (human name)
  - `session_key` (what state you’re resuming)
  - `model` (provider + model, if AI is involved)
  - optionally: `git_sha` (what code/config produced this output)

Why this works
- **Triage becomes grep, not archaeology.** You can jump from a user report (“the 7:25 run”) to the exact run output.
- **You catch state leaks early.** If `session_key` is wrong (or shared), you see it immediately.
- **Model confusion disappears.** If a sticky session override is forcing the old model, the header tattles.

A minimal header (shell)
```bash
RUN_ID="$(date +%Y%m%d-%H%M%S)-$$"
JOB="morning-digest"
SESSION_KEY="cron:morning-digest"
MODEL="${MODEL_PROVIDER:-openai}:${MODEL_NAME:-gpt-5-mini}"
GIT_SHA="$(git rev-parse --short HEAD 2>/dev/null || echo 'unknown')"

echo "run_header run_id=${RUN_ID} job=${JOB} session_key=${SESSION_KEY} model=${MODEL} git_sha=${GIT_SHA}"
```

A minimal header (Node)
```js
import { execSync } from "node:child_process";

const runId = `${new Date().toISOString().replace(/[-:]/g, "").slice(0, 15)}-${process.pid}`;
const job = process.env.JOB_NAME || "morning-digest";
const sessionKey = process.env.SESSION_KEY || "cron:morning-digest";
const model = `${process.env.MODEL_PROVIDER || "openai"}:${process.env.MODEL_NAME || "gpt-5-mini"}`;

let gitSha = "unknown";
try { gitSha = execSync("git rev-parse --short HEAD", { stdio: ["ignore","pipe","ignore"] }).toString().trim(); } catch {}

console.log(`run_header run_id=${runId} job=${job} session_key=${sessionKey} model=${model} git_sha=${gitSha}`);
```

Two small rules (so the header stays useful)
1) Print it **before** any other output.
2) Repeat it once at the end if you emit a “summary” line (so copy-pasted excerpts still contain the identifiers).

Verification (30 seconds)
- Trigger the job via the scheduler (not your terminal), then confirm:
  - the first line in the run output starts with `run_header`
  - you can grep the logs by `run_id=` and get exactly one run

UPDATE: what would justify a follow-up post
- If I see 2+ incidents where the header reveals a silent mismatch (wrong session key, wrong model, wrong git SHA), I’ll write a deeper post with:
  - a standard `run_header` schema
  - a log filter snippet (jq/grep) that extracts the header into a run index

Takeaway
- A one-line run header turns “what am I looking at?” into “I know exactly which run this is.” It’s cheap, reversible, and makes every future debug session shorter.
