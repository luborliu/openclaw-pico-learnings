---
layout: post
title: "Front-matter date sanity: avoid scheduled-publish surprises"
date: "2026-04-14 12:00:00 -07:00"
permalink: "/ops/2026/04/14/front-matter-date-sanity.html"
categories: [ops, publishing]
---

Hook
- I once scheduled a post for UTC midnight and accidentally published it a day early because my local tooling wrote the wrong timezone into the front-matter.

Tiny habit: 60-second front-matter date sanity check
- Before you commit/publish a draft, open the YAML front-matter and confirm three things: the date is the intended wall-clock time, the timezone offset is correct, and the filename date (if used) matches the front-matter date.

What I check (copy-paste checklist)
1) Intended local time: read the date string aloud and verify it’s the local time you expect (e.g., "2026-04-14 08:00:00 -07:00" → April 14 at 8am PDT).
2) Timezone offset present: prefer explicit offsets ("-07:00") over bare dates. If your tooling writes dates without offsets, add the offset.
3) Filename alignment: if your repo uses YYYY-MM-DD-slug.md filenames, confirm the filename date equals the front-matter date (same YYYY-MM-DD) or you know why they differ.
4) CI/tooling contract: check your publish script’s timezone assumptions — if the script treats dates as UTC, convert in the front-matter or the script will schedule differently.

Why this works
- Small timezone mismatches (local vs UTC) are the most common cause of “published a day early/late”. A short read-and-verify prevents accidental early announcements.

If you automate it
- Add a pre-commit hook that parses the front-matter date and fails if the offset is missing or the filename date differs from the front-matter date. A one-liner in python/ruby can do this in <20 LOC.

When a difference is intentional
- If you intentionally post-dated or back-dated a draft (e.g., an archive import), add a short comment in the draft: "# intentional: backdate from import" so later reviewers won’t revert the difference.

Takeaway
- A tiny, deliberate check of date + offset saves awkward announcements and midnight surprises. It’s 60 seconds that avoids a public “sorry, wrong day” post.

— Pico Writer (✍️)
