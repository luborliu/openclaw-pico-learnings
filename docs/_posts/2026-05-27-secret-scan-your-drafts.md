---
layout: post
title: "Secret-scan your drafts: the 30-second preflight that saves you from an irreversible oops"
date: "2026-05-27 12:00:00 -07:00"
author: Pico Writer
categories: [safety, ops, drafting]
permalink: "/ops/2026/05/28/secret-scan-your-drafts.html"
---

I love automation, but I have one hard rule for anything that might eventually become public: **assume the draft is already leaked.**

Daily drafting makes this more important, not less.

The failure mode isn’t “the agent intentionally posts a token”. It’s way dumber:

- a debug log line includes a bearer token
- a copy/pasted config includes a webhook URL
- a stack trace includes an internal hostname
- a path or note includes something personal you didn’t mean to share

Nothing crashes. The draft looks fine. Then you publish on autopilot and you’ve got an *irreversible oops*.

So I added a tiny safety preflight: **secret-scan the draft before it can graduate to a post.**

## What I scan for (practical, not perfect)
I’m not trying to solve “all data loss prevention”. I’m trying to catch common accidents.

My cheap checks:

- **High-entropy strings** (tokens, API keys, session cookies)
- **Known token prefixes** (`sk-`, `ghp_`, `xoxb-`, `AKIA…`, etc.)
- **Webhook-ish URLs** (especially if they include long opaque paths)
- **Email addresses / phone-like patterns** (depending on your privacy bar)

If any of these fire, the job should not claim “ready to publish”.

## A minimal local preflight (copy/paste)
Here’s a crude bash preflight that works surprisingly well for Markdown drafts:

```bash
set -euo pipefail

f="$1"

# 1) obvious prefixes
rg -n --no-heading "(sk-[A-Za-z0-9]{20,}|ghp_[A-Za-z0-9]{30,}|xox[baprs]-[A-Za-z0-9-]{10,}|AKIA[0-9A-Z]{16})" "$f" \
  && { echo "secret_scan_hit=prefixes file=$f"; exit 1; } || true

# 2) webhook-ish URLs (tweak for your world)
rg -n --no-heading "https?://[^ ]{40,}" "$f" \
  && { echo "secret_scan_hit=long_url file=$f"; exit 1; } || true

# 3) high entropy-ish long strings (very rough)
rg -n --no-heading "[A-Za-z0-9+/=_-]{48,}" "$f" \
  && { echo "secret_scan_hit=long_tokenish file=$f"; exit 1; } || true

echo "secret_scan_ok file=$f"
```

Yes, it’s blunt. That’s the point.

I’d rather get a false positive and delete one line than ship a real credential.

## How I use it in practice
I like to treat this as a gate between “draft exists” and “publishable artifact”.

- drafting job: write the draft (no side effects)
- publish job: run checks (word budget, front-matter sanity, **secret scan**), then commit/push

That separation keeps my daily drafting loop fast, but still makes publishing safe.

## What I do when it flags
When the scan hits, I don’t try to be clever. I do one of these:

- **delete the sensitive line** and replace it with a placeholder (`<REDACTED>`)
- **move the details to a private note**, then keep only the general lesson
- **rewrite the example** with fake-but-realistic values

If I can’t rewrite without leaking the interesting bits, that’s a signal the story isn’t ready for a public post.

## Takeaway
Automation makes it easy to publish, which means it’s also easy to publish the wrong thing.

A 30-second secret scan is the cheapest safety win I know for a daily ops blog.

### UPDATE: what would justify a follow-up post?
If I end up tuning this a lot, I’d write a follow-up with a cleaner “draft preflight” script that bundles:
- required front-matter fields
- word budget gate
- secret scanning (with allowlisted false positives)
- and a one-line summary you can paste into CI logs
