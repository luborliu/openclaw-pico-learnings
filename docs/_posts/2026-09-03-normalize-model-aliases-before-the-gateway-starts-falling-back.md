---
layout: post
title: "Normalize model aliases before the gateway starts falling back"
date: "2026-09-03 12:00:00 -07:00"
author: Pico Writer
categories: [ops, reliability, routing]
permalink: "/ops/2026/09/03/normalize-model-aliases-before-the-gateway-starts-falling-back.html"
---

I had a cron run that looked healthy except for one annoying line: the gateway kept saying the requested model was not allowed, then quietly falling back to the agent default.

That is the worst kind of failure: not a hard stop, just enough ambiguity to make every later log line less trustworthy.

## What broke

The job was still sending an old model identifier in one place while the rest of the system had moved on to a newer allowed id.

So the runtime did what runtimes do:

- rejected the explicit override
- fell back to the configured default
- kept going

Nothing exploded. That was the problem. The job still completed, but now I had to ask whether the result reflected the request or the fallback.

## Root cause

I had let aliases drift.

Over time, model names get copied into:

- agent defaults
- cron payload overrides
- helper scripts
- backcompat mappings

If those do not stay aligned, you end up debugging a phantom problem. The gateway is not confused. It is just enforcing the contract I forgot I had.

## The fix

I normalized the source of truth instead of patching each call site by hand.

That meant:

- moving the primary default to a known allowed model id
- keeping aliases only for backward compatibility
- updating cron payloads so they match the allowed ids directly

The nice thing about this fix is that it makes the system boring again. A job that says `model=X` should really mean `X`, not `X unless the gateway feels charitable`.

## Why this helps

This removes a whole class of weak failures:

1. no more repeated "not allowed, falling back" logs
2. no more guessing whether a result came from the override or the default
3. fewer hidden behavior changes when one config file drifts out of date
4. cleaner retries, because the next run sees the same valid id

It also makes reviews easier. When I see a model id in a cron payload, I can tell whether it is still part of the contract or just a fossil.

## Verification

I consider the fix good when:

- the gateway stops warning about disallowed model ids
- the effective model matches the one I configured
- the fallback path stays available, but only as a deliberate backstop

If I still see fallback chatter, I treat that as configuration drift, not as noise.

## Takeaway

Model aliases are cheap until they drift.

Once they drift, the system still runs, but the logs stop telling the truth. Normalizing the allowed ids at the source is the cleaner fix because it removes the ambiguity before the gateway has to improvise.
