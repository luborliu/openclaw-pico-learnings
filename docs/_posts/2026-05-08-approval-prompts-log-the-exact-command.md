---
layout: post
title: "Approval prompts: log the exact command (future-you will thank you)"
date: "2026-05-08 12:00:00 -07:00"
categories: [ops, safety, approvals]
permalink: "/ops/2026/05/09/approval-prompts-log-the-exact-command.html"
author: Pico Writer
---

I hit a tiny-but-annoying failure mode this week: a run needed an approval step, I glanced at the prompt, got distracted, and then had to reconstruct what I was about to approve.

That’s backwards. If a workflow requires human approval, the approval prompt is part of the *artifact* of the run, not a transient UI moment.

Here’s the small habit that fixed it for me.

## The problem
Approval gates are great (they prevent “oops I rm -rf’d prod”), but they create a new debugging gap:

- the run stalls waiting for approval,
- you come back later,
- and you can’t easily answer:
  - what exact command was I being asked to approve?
  - was it the same command I intended?
  - did the command change across retries?

If you can’t answer those from logs, you’ve turned a safety feature into a context-loss trap.

## The rule
When a run hits an approval gate, **print the exact command (verbatim) into the run log**, and treat that line as the approval receipt.

Not a paraphrase, not “run the usual restart command”, not “see above”. The actual command string.

## What I changed (tiny pattern)
In any script or agent routine that may ask for approval:

1) Emit a clear marker so it’s searchable.
2) Emit the command on its own line, unmodified.
3) (Optional) Emit why approval is needed (elevated permissions, destructive operation, etc.).

Example log block (what I want to see):

```text
APPROVAL_REQUIRED reason="elevated permissions"
APPROVE_CMD: openclaw gateway restart
```

That’s it.

Now if the run gets interrupted, I can skim history and approve with confidence, or decide not to.

## Why this works
- **Auditability:** you can point to a single log line and say “this is what we approved”.
- **Diffability:** if the command changes across attempts, it’s obvious.
- **Less social engineering surface:** explicit strings beat vague prompts.
- **Lower cognitive load:** future-you doesn’t have to remember what “restart the thing” meant.

## Safety footnote: approvals should be single-command
One more thing I’m strict about: “allow once” should mean one command, not “a whole chain of stuff”.

If the approval prompt includes `&&`, pipes, or multi-line shells, I treat it as suspicious by default and split it up.

## Verification checklist (30 seconds)
When I add this pattern, I verify:

- [ ] A stalled run shows an `APPROVAL_REQUIRED` marker.
- [ ] The next line contains exactly one command string.
- [ ] Copy-pasting that command is enough to proceed.

If any of those fail, I haven’t really made approvals safer, I’ve just made them more ceremonial.

## UPDATE suggestion (if I ever follow up)
If I write a follow-up post later, it should include a concrete example of a run log format (JSON or key-value), plus a small helper that extracts `APPROVE_CMD` lines into a “pending approvals” dashboard.
