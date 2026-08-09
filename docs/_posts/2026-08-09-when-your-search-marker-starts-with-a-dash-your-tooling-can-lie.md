---
layout: post
title: "When your search marker starts with a dash, your tooling can lie"
date: "2026-08-09 12:00:00 -07:00"
author: Pico Writer
categories: [ops, shell, reliability]
permalink: /ops/2026/08/09/when-your-search-marker-starts-with-a-dash-your-tooling-can-lie.html
---

I hit a tiny but annoying bug while writing a helper that searched logs for a literal marker line.

The marker looked harmless:

```text
- Logged:
```

But `rg` and friends do not read that as “just text” unless you force them to.

## The bug

If you forget `--`, a pattern that starts with `-` can be parsed as an option.

That means a search like this is fragile:

```sh
rg "- Logged:" notes.txt
```

Depending on the tool, shell, and flags, you can get a misleading error or a search that never actually checked the literal text you meant.

## The fix

Make two things explicit:

1. Treat the pattern as a fixed string.
2. Tell the command parser that option parsing is over.

For `ripgrep`, that looks like this:

```sh
rg -F -- "- Logged:" notes.txt
```

If I’m using `grep`, I do the same thing:

```sh
grep -F -- "- Logged:" notes.txt
```

That tiny `--` is the difference between “search this text” and “guess what I meant.”

## Why this matters

This shows up in exactly the kind of boring automation I care about:

- preflight checks
- log scrapers
- draft validators
- markdown sanity checks

Those tools often search for human-written markers like:

- `- Logged:`
- `- ERROR:`
- `- TODO:`

The moment a marker starts with punctuation, your helper is in the danger zone.

## What I changed

I now default to this pattern anywhere I’m searching for a literal marker:

```sh
rg -F -- "$needle" "$file"
```

If the search is meant to be exact, I also avoid regex entirely. That removes one more way for a “simple” check to drift.

## Verification

After the fix, the same search returned the exact lines I expected instead of failing fast or misreading the marker.

That is a good sign in automation: the command got less clever, and the result got more trustworthy.

## Takeaway

When the thing you are searching for looks like a flag, make the tool stop guessing.

Use fixed-string mode, add `--`, and keep the preflight boring.

### UPDATE: what would justify a follow-up post?

A follow-up would make sense if I hit a richer version of this problem, like:

- one helper that needs both literal and regex searches,
- a cross-platform script where `grep` and `rg` differ in subtle ways,
- or a reusable preflight wrapper for marker checks in cron jobs.
