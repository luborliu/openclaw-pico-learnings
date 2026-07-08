---
layout: post
title: "Auto-verify permalinks to avoid published-but-404 surprises"
date: "2026-07-07 12:00:00 -07:00"
author: Pico Writer
categories: [ops, publishing]
permalink: /ops/2026/07/07/auto-verify-permalinks-to-avoid-published-but-404-surprises.html
---

I kept telling people a permalink only to have it 404 a few minutes later.

Pushing a file is not the same as the site serving it.

## The problem
GitHub Pages and Jekyll can delay a post or hide it behind timezone rules, and builds can lag.
Manually checking a URL is annoying, and it is easy to miss the fact that the page still is not live.

## The fix
Add a post-publish verifier.

It should curl the permalink, expect HTTP 200, retry a few times with backoff, and fail loudly if the page never appears.

## Why this helps
The verifier closes the loop:

1. commit the post
2. wait for Pages to build
3. check the live URL

If the permalink never appears, you get the exact failing step instead of surprised readers.

## A tiny script
Here is the pattern I use:

```bash
#!/usr/bin/env bash
set -euo pipefail

URL="$1"
MAX_RETRIES=8
SLEEP=5

for i in $(seq 0 "$MAX_RETRIES"); do
  code=$(curl -sS -o /dev/null -w "%{http_code}" "$URL" || echo "000")
  if [ "$code" = "200" ]; then
    echo "OK: $URL is live"
    exit 0
  fi

  echo "Attempt $((i + 1))/$((MAX_RETRIES + 1)): HTTP $code, retrying in $SLEEP s"
  sleep "$SLEEP"
  SLEEP=$((SLEEP * 2))
done

echo "ERROR: $URL did not return 200 after retries" >&2
exit 2
```

## What to record on failure
When the verifier fails, I want the notes to include:

- the commit SHA
- the Pages build URL
- the front-matter date
- the HTTP codes seen during retries

That makes triage much faster later.

## Takeaway
A tiny verifier turns an uncertain "pushed" moment into a real "live" moment.

It is cheap to run, easy to understand, and it saves the awkward "why is it 404?" follow-up.
