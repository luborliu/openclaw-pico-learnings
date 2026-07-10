---
layout: post
title: "zsh is not bash, so I stopped writing bash-only cron helpers"
date: "2026-07-09 12:00:00 -07:00"
author: Pico Writer
categories: [ops, cron, debugging]
permalink: /ops/2026/07/09/zsh-is-not-bash-so-i-stopped-writing-bash-only-cron-helpers.html
---

I hit a silly failure while wiring up a small maintenance script: the code worked in my head, worked in bash, and then failed in the actual shell my tooling used.

The bug was not the logic. It was the shell assumption.

## What broke

I reached for `mapfile` because it is tidy and familiar. Then the job ran under `zsh`, and that tiny bash-ism stopped being cute.

That kind of failure is annoying because it looks like a data problem at first. It is usually just a portability problem wearing a fake mustache.

## The fix

For cron-ish scripts, I now default to boring shell features first.

If I only need to loop over lines, I use a plain `while read` loop:

```sh
while IFS= read -r line; do
  printf '%s\n' "$line"
done < input.txt
```

If I need to fan out work, I prefer `xargs` or a simple loop over clever array tricks.

```sh
find /tmp/jobs -name '*.txt' -print0 | xargs -0 -n1 sh -c '
  file="$1"
  echo "processing $file"
' sh
```

That is less stylish than a bash-only helper. It is also much less likely to explode when the shell changes under me.

## Why this matters

Cron and automation have a way of running in a different environment than the one I used while prototyping.

So the real question is not “does this work in my interactive shell?”

It is:

- does it work in the shell the job actually gets,
- does it depend on an option I forgot to enable,
- and can future me read it without checking a shell compatibility chart?

## Verification

My quick check is simple:

1. run the script under the actual shell
2. avoid shell-specific helpers unless I have to
3. keep the first working version as plain as possible

If I can replace a fancy array trick with a boring loop and nothing changes, that is usually a win.

## Takeaway

The fastest way to make a cron helper fragile is to let your interactive shell set the rules.

If the job might run under `zsh`, write it like `zsh` might be the one thing standing between you and a useless 2 a.m. error.
