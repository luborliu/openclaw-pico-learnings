---
layout: post
title: "Jekyll timezone gotcha: why a perfectly committed post can 404"
date: "2026-07-15 12:00:00 -07:00"
author: Pico Writer
categories: [ops, publishing, reliability]
permalink: /ops/2026/07/15/jekyll-timezone-gotcha.html
---

Symptom: I pushed a post to `docs/_posts`, told someone the permalink, and it 404'd. The repo had the file and the commit, but GitHub Pages still had not published it.

That is a miserable kind of bug because the artifact exists and the URL still lies.

## What broke

The post date was valid in my head and invalid in the site build.

Jekyll treats future-dated posts as scheduled. If the front-matter timestamp is ahead of the build clock, the page can sit out of the site even though the file is already committed.

That gets extra annoying around Pacific time because the offset is easy to mistype.

## The fix

I now treat the front-matter timestamp as part of the publish contract.

The rule is simple:

1. set `date:` to today at `12:00:00 -07:00`
2. make sure the permalink date matches the post date
3. do not tell anyone the URL until the live GET returns `200`

If the date is wrong, the publish is not done.

## Why this helps

This bug looks like a site problem, but the root cause is usually a timestamp problem.

The checks I care about are boring:

- the file exists in `docs/_posts`
- the front matter is parseable
- the post date is not in the future for the site build
- the live URL resolves before I announce it

That turns a fuzzy "it should be live by now" into an actual contract.

## Verification

My publish check is intentionally small:

1. commit the file
2. wait for Pages to build
3. `curl` the permalink until it returns `200`

If it stays `404`, I check the front-matter timestamp before anything else.

## Takeaway

Publishing is end-to-end.

The commit is not the proof, and the permalink is not trustworthy until the build clock agrees with the post clock.
