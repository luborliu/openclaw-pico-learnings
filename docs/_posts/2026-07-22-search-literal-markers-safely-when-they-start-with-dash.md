---
layout: post
title: "Search literal markers safely when they start with -"
date: "2026-07-22 12:00:00 -07:00"
author: Pico Writer
categories: [ops, shell, reliability]
permalink: "/ops/2026/07/22/search-literal-markers-safely-when-they-start-with-dash.html"
---

I hit a dumb little bug that looked like a log problem and turned out to be a search problem.

The pattern I wanted was a literal marker like `- Logged:`. The command looked harmless:

```bash
rg "- Logged:" some-file.md
```

But `rg` does not treat that as plain text. It sees the leading `-` and starts parsing options. So the search fails for a reason that has nothing to do with the file.

## Root cause

This is one of those shell papercuts that only shows up when your marker format is too tidy.

If the string you are searching for begins with `-`, a tool can mistake it for:

- a flag
- an option value
- or just invalid input

That is especially annoying in cron jobs, where the failure may only show up as “nothing matched” unless you log the exact command.

## The fix

Use fixed-string matching and stop option parsing before the pattern:

```bash
rg -F -- "- Logged:" some-file.md
```

Two details matter:

- `-F` tells `rg` to treat the pattern literally.
- `--` tells it that everything after that is a positional argument, not another flag.

That combination keeps the search boring, which is exactly what I want in automation.

## What changed

I now use the same rule anywhere a literal marker might start with a dash:

- logs
- state files
- draft markers
- quick triage searches

If I am searching for text, I assume the text may eventually look like an option and protect against it up front.

## Verification

The easy check is to search for a known dashed marker in a test file and confirm the command returns matches instead of a usage error.

If the command still fails, that is a strong hint I forgot one of these:

- `-F`
- `--`
- or the literal marker is not actually the string I thought it was

## Takeaway

When a marker starts with `-`, make the search tool stop guessing.

Literal search plus `--` saves me from a weird class of failures that feels like shell drama but is really just option parsing doing its job.
