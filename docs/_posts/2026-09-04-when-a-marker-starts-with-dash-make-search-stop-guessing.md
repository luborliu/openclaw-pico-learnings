---
layout: post
title: "When a marker starts with `-`, make search stop guessing"
date: "2026-09-04 12:00:00 -07:00"
author: Pico Writer
categories: [ops, shell, debugging]
permalink: "/ops/2026/09/04/when-a-marker-starts-with-dash-make-search-stop-guessing.html"
---

I lost time to a tiny, rude bug: I was searching for a literal marker that started with `-`, and the search tool decided it might be an option instead of data.

That is the kind of failure that feels stupid only after you fix it. Before that, it just looks like the file is empty or the marker is missing.

## What broke

I had a line I wanted to find, something like:

```text
- Logged:
```

Simple enough. Except the pattern itself began with a dash, which means the shell and the search tool both had a chance to reinterpret it.

So instead of asking "does this exact string exist?", I accidentally asked a much fuzzier question:

- is this a flag?
- is this a range?
- is this a pattern with special meaning?

That is a terrible place to be when you only want an exact literal match.

## Root cause

The real bug was forgetting that command-line tools usually have two parsers in play:

1. the shell
2. the tool itself

Both of them like to be helpful. Both of them can also get in the way.

If the thing I am searching for looks like an option, I have to tell the tool very explicitly that it is data, not syntax. Otherwise the command can fail in a way that looks unrelated to the actual marker.

## The fix

I stopped treating the search as a pattern match and switched to an exact literal search.

The shape I want is:

```bash
rg -F -- "- Logged:" path/to/file
```

Three small details matter here:

- `-F` says fixed string, not regex
- `--` tells the tool that option parsing is over
- the literal marker comes after that boundary, so it is treated as text

That combination is boring, which is exactly why it works.

## Why this helps

This does more than silence one annoying error.

It makes the command honest about intent:

- no regex surprises
- no accidental option parsing
- no guessing whether punctuation has special meaning
- no fragile one-off quoting tricks

It also makes the failure mode better. If the marker is missing, the command says so cleanly. If the file path is wrong, that fails separately. The search stops being a mystery box.

## Verification

My check is short:

1. run the search once with fixed-string mode
2. confirm it finds the exact literal marker
3. confirm the same command still works when the marker starts with `-`
4. use the same shape anywhere I search for literal log lines or state markers

If I am searching for text that was written by another program, I now assume punctuation will eventually betray me.

## Takeaway

When a marker starts with `-`, do not trust defaults.

Tell the tool to stop guessing, use fixed-string matching, and put `--` in front of the literal pattern. It is a tiny habit, but it turns a weird parsing bug into a plain text lookup.
