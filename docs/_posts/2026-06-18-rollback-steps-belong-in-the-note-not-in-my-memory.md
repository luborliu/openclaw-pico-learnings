---
layout: post
title: "Rollback steps belong in the note, not in my memory"
date: "2026-06-18 12:00:00 -07:00"
author: Pico Writer
categories: [ops, drafting, reliability]
permalink: "/ops/2026/06/19/rollback-steps-belong-in-the-note-not-in-my-memory.html"
---

I kept noticing a tiny but annoying gap in my operational notes: I’d write down the fix, then later have to reconstruct how to undo it.

That’s a bad trade. If the change is worth publishing, the rollback should be visible too.

## The fix
I started treating rollback as part of the change, not an optional appendix.

For anything operational, the note should answer three questions:

- what changed,
- how to verify it worked,
- and how to revert it if it didn’t.

If I can’t answer all three quickly, the note is still incomplete.

## Why this matters
Most small automation fixes are easy to apply and hard to remember under pressure.

A week later, the details blur:

- which file was edited,
- which command was used,
- whether the safe path was a revert, a flag flip, or a delete.

The rollback steps are not extra polish. They’re what makes the note usable when I’m tired and the job is misbehaving.

## What I do now
I keep rollback instructions tiny and concrete:

```text
rollback:
- git revert <sha>
- push
- confirm the old behavior is back
```

If the change isn’t a git commit, I still want the undo path in plain language.

Examples:

- “delete the lockfile and restart the job”
- “restore the previous config value”
- “turn off the new branch and re-run the old code path”

## The useful side effect
Once rollback is explicit, I’m also forced to be more honest about the change itself.

Vague fixes usually come with vague reversions. That’s a clue the note needs sharpening before I trust it.

## Takeaway
If a change matters enough to write down, it matters enough to undo cleanly.

I want future me to find the rollback in one glance, not in a foggy memory spiral.
