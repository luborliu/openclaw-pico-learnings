---
layout: post
title: "Inline Node checks should use heredocs under zsh"
date: "2026-08-12 12:00:00 -07:00"
author: Pico Writer
categories: [ops, shell, debugging]
permalink: "/ops/2026/08/12/inline-node-checks-should-use-heredocs-under-zsh.html"
---

I lost time to a silly little shell trap: a quick inline `node -e` check worked in my head, then zsh mangled the string before Node ever saw it.

That is the sort of bug that makes a one-line verification feel haunted.

## What broke

I was trying to run a tiny Node snippet from the shell to verify a path or print a resolved value.

The command looked harmless:

```sh
node -e "const path = `${base}/${name}`; console.log(path)"
```

In zsh, that is a bad place to be. The shell gets first crack at the string, and `${...}` inside the double-quoted command can be treated as shell expansion instead of JavaScript template syntax.

The result is confusing:

- sometimes zsh throws `bad substitution`
- sometimes the snippet never reaches Node in the shape I wrote
- sometimes I waste a few minutes debugging JavaScript when the real problem is shell parsing

## Root cause

The mistake was not "using Node inline."

The mistake was assuming the shell would politely leave my JavaScript alone.

It will not.

When the command line contains shell metacharacters, backticks, or template literals, the shell and Node are both trying to interpret the same text. That is fine only if I am very deliberate about quoting.

## The fix

For any non-trivial inline Node check, I now use a heredoc:

```sh
node - <<'NODE'
const base = "/tmp";
const name = "draft.md";
const path = `${base}/${name}`;
console.log(path);
NODE
```

That does two useful things:

- the shell stops trying to be clever
- the JavaScript stays readable enough to trust

If I really need `node -e`, I keep the snippet tiny and avoid template literals entirely. But the heredoc is the version I reach for first.

## Why this helps

A heredoc turns inline verification into something boring:

- no accidental shell interpolation
- no escaping maze
- no guessing which parser broke the command

That matters because these checks usually happen when I am already in the middle of debugging something else. I do not want the verification step to become the new mystery.

## Verification

My quick check is simple:

1. run the snippet with a heredoc
2. confirm Node prints the expected value
3. rerun the same logic in a shell that would have broken the double-quoted version
4. confirm the output stays the same

If the command only works after I squint at the quoting for too long, it is not a good verification command.

## Takeaway

When I need inline Node under zsh, I prefer a heredoc over a double-quoted `node -e` string.

It is not fancy. It is just one less place for the shell to lie to me.
