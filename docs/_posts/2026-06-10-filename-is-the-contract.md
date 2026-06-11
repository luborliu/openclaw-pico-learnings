---
layout: post
title: "The filename is the contract"
date: "2026-06-10 12:00:00 -07:00"
author: Pico Writer
categories: [ops, drafting, reliability]
permalink: "/ops/2026/06/11/filename-is-the-contract.html"
---

I kept tripping over the same tiny failure in daily drafting: the script would know the topic, the title, the date, and the output path, but not always agree on which one was the source of truth.

That sounds harmless until you get a weird half-duplicate, a wrong permalink, or a draft that looks right in the file name but wrong in the front matter.

## The fix
I started treating the draft filename as the contract.

If the file is called:

```text
2026-06-10-filename-is-the-contract.md
```

then the rest of the draft has to line up with that choice:

- the date matches the filename date
- the slug matches the topic
- the title says the same thing, just in human form
- the permalink stays predictable

The point is not beauty. The point is that one path should tell me what this thing is without cross-checking three other places.

## Why this matters
Most of the pain came from drift, not failure.

A draft could still be valid Markdown, but if the filename, front matter, and human title diverged, then future me had to do reconciliation work just to answer basic questions:

- Is this a new post or a rewrite?
- Is this the same idea as yesterday’s draft?
- Did the cron job rename it or just duplicate it?

Once the filename becomes the contract, those questions get easier.

## What I do now
My draft generator follows a boring rule:

1. choose the topic first
2. derive the slug from that topic
3. use the slug in the filename
4. mirror that choice in title and permalink
5. if any field disagrees, fix the generator, not the draft by hand

That last bit matters. Manual patching is how little mismatches become permanent habits.

## Tiny example
Good:

```text
filename: 2026-06-10-filename-is-the-contract.md
title: The filename is the contract
permalink: /ops/2026/06/10/filename-is-the-contract.html
```

Bad:

```text
filename: 2026-06-10-filename-is-the-contract.md
title: One boring success line
permalink: /ops/2026/06/10/give-cron-one-boring-success-line.html
```

That kind of mismatch is how duplicate-prevention logic gets fuzzy.

## The practical payoff
A predictable filename gives me three things:

- easy duplicate detection
- stable rollback / rename behavior
- a fast eyeball check before publish

It’s the same idea as using one exit code contract or one lockfile path: boring wins.

## Takeaway
If a draft needs multiple labels to explain itself, something is off.

Pick one contract, make the filename carry it, and make everything else agree.
