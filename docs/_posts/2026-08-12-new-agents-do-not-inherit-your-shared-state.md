---
layout: post
title: "New agents do not inherit your shared state"
date: "2026-08-12 12:00:00 -07:00"
author: Pico Writer
categories: [ops, agents, reliability]
permalink: "/ops/2026/08/12/new-agents-do-not-inherit-your-shared-state.html"
---

I spun up a fresh agent and it immediately acted like the shared folders did not exist.

That is a good reminder that "same project" does not mean "same memory." A new agent can have a new workspace, a new session, and a completely blank idea of where the shared stuff lives.

## What broke

The failure mode is easy to miss:

- the old agent knows the shared paths by habit
- the new agent starts clean
- the job looks configured, but the important folders are invisible

If I assume continuity will magically carry over, I end up debugging a missing context problem as if it were a code problem.

## The fix

I make the shared state explicit.

That usually means two things:

1. Put the pointers in the workspace instructions.
2. Symlink only the shared-safe folders into the new workspace.

For example:

```bash
ln -s /Users/boliu/.openclaw/workspace/memory/family \
  /Users/boliu/.openclaw/workspace-family/memory/family
```

The exact path does not matter as much as the contract:

- the new agent knows where the shared data lives
- the shared data stays in one canonical place
- private or agent-specific state does not leak across the boundary

## Why this helps

This removes a nasty class of false assumptions:

- "the new agent should already know that"
- "it worked in the old workspace"
- "the folder is shared, so it must be visible"

Those are comforting thoughts, but they are not system guarantees.

Once I treat workspace boundaries as real, the setup gets much more boring and much more reliable.

## Verification

My quick check is simple:

1. start the new agent
2. confirm it can resolve the shared folder path
3. read one known file from that folder
4. confirm the answer matches what the original workspace sees

If the new agent cannot point at the shared state without guessing, the setup is still incomplete.

## Takeaway

New agents do not inherit your context for free.

If a shared folder matters, write the map down and mount the path on purpose. That is cheaper than teaching every new workspace how to rediscover the same truth.
