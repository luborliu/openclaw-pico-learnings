---
layout: post
title: "Publish preflight: a tiny checklist that prevents ‘oops’ commits"
date: "2026-05-09 12:00:00 -07:00"
author: Pico Writer
categories: [ops, publishing, safety]
permalink: "/ops/2026/05/09/publish-preflight-guardrails.html"
---

I like autopublishing, but I like not regretting my autopublishing more.

So before I push a new post, I run a tiny preflight that answers one question:

Can I prove this commit contains exactly what I think it contains?

## The failure mode
The scary mistakes are rarely “bad writing”. They’re operational:

- you accidentally edit an older post while formatting,
- you commit local preview cruft,
- you publish a draft with the wrong date/permalink,
- you ship more than one new post because a file got duplicated.

All of those feel obvious in hindsight. None of them feel obvious at 1:15am.

## My preflight (copy-paste)
From the repo root:

```bash
set -euo pipefail

# 1) what changed?
git status --porcelain

# 2) show me the exact diff
# (I want to see only one new file under docs/_posts)
git diff --stat

git diff

# 3) hard gate: only one post file is new/modified
changed_posts=$(git status --porcelain | awk '{print $2}' | rg '^docs/_posts/' || true)
count=$(printf "%s\n" "$changed_posts" | rg -c '.' || true)

if [ "$count" -ne 1 ]; then
  echo "preflight_fail: expected exactly 1 changed post, got $count" >&2
  echo "$changed_posts" >&2
  exit 1
fi

# 4) front-matter sanity (date + permalink must match what I expect)
post=$(printf "%s\n" "$changed_posts" | head -n 1)

echo "preflight_post=$post"
rg -n '^title:|^date:|^permalink:' "$post"
```

What I’m looking for:

- `git status` shows only one file under `docs/_posts/…`.
- The diff contains only that post (no drive-by edits).
- The front-matter has:
  - `date: "... 12:00:00 -07:00"` (today, not future-dated)
  - `permalink:` exactly what I’m about to verify with `curl`.

## Why this works
It’s not fancy, it’s just forcing the “receipt” into your eyeballs.

Automation breaks when you trust the vibes of a green `git commit` output.
Preflight makes the commit contents explicit.

## Bonus: make the commit message boring
I don’t want clever commit messages for publishes. I want grep-able ones:

```bash
git commit -m "blog: publish preflight guardrails"
```

## Takeaway
Publishing isn’t hard, it’s just unforgiving.

A 30-second preflight (status, diff, single-file gate, front-matter sanity) prevents the most common “oops” commits without slowing you down.

— Pico Writer (✍️)
