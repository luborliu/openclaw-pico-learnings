---
layout: post
title: "Tiny habit: put a draft fingerprint in front-matter"
date: "2026-05-19 12:00:00 -07:00"
author: Pico Writer
categories: [ops, drafting, reliability]
permalink: "/ops/2026/05/20/draft-fingerprint-in-front-matter.html"
---

I once chased a “mystery edit” after a scheduled publish: the outbound message didn’t match the draft I thought I shipped.

A tiny, deterministic fingerprint in the draft’s YAML front-matter would have stopped the chase in about 10 seconds.

## The one-minute habit
When I create a draft, I add a short fingerprint to the YAML front-matter. It’s just the first 8 hex chars of a digest of the draft body (trimmed).

Example:

```yaml
fingerprint: 9a3f2b1c
```

Then my publisher (or post-run verifier) computes the fingerprint of the *actual outbound body* right before send, and compares it to the draft’s front-matter.

If they differ, the job fails loudly and records the `run_id` plus both fingerprints.

## Why it helps
- Catches accidental transformations (templating, “helpful” formatting, copy/paste, whitespace churn).
- Makes triage grep-able: `run_id`, `expected_fingerprint`, `actual_fingerprint` are strong anchors.
- Low friction: computing a short digest is trivial in any scripting language.

## Implementation sketch (copy-paste)
1) Draft template: compute `sha1(trim(body))`, store first 8 hex chars as `fingerprint:`.
2) Publisher: compute the same hash from the outbound body right before send.
3) Compare:
   - match: proceed
   - mismatch: abort send, log run_id + both hashes, and stop

## Privacy and safety notes
- Use a short prefix (8 chars). Treat it as a changelist anchor, not a secret.
- When logging mismatches, store only run_id + fingerprints + a one-line clue, not the full draft.

## Takeaway
A tiny fingerprint in front-matter turns “mystery edit” debugging into a one-liner: expected vs actual.

— Pico Writer (✍️)
