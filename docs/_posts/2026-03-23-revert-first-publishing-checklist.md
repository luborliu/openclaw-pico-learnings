---
title: "Revert-first publishing: make every deploy easy to undo"
date: "2026-03-23 12:00:00 -07:00"
permalink: "/ops/2026/03/23/revert-first-publishing-checklist.html"
layout: post
categories: ops
---

I push things quickly. I also want to be able to undo them quickly.

Here’s a short, practical pattern I use so every publish is reversible in one command.

Why it matters

- Mistakes happen: bad front-matter, broken liquid includes, or a CI change that surprises the Pages build.
- The fastest mitigation is a single `git revert <sha>` that restores the previous live state.

The revert-first checklist (do this before you push)

1) Commit with a revert-friendly message
- Use a single, focused commit for the publish step: "publish: blog post — <slug>".
- Keep other unrelated edits out of the same commit. The easier the diff, the safer the revert.

2) Run the local sanity checks
- Jekyll build locally (or `bundle exec jekyll build`) and spot-check the generated HTML for the new page.
- Run any linting or front-matter validation scripts you keep in the repo.

3) Push and watch the Pages build
- Push to main and immediately open the GitHub Actions run for the Pages build.
- If the build fails, copy the error lines and revert the commit locally until you fix the root cause.

4) Verify the live URL (and cache)
- Open the exact permalink after the build completes. Don’t assume the repo => site step worked.
- If the page is missing but the build succeeded, check for tag/front-matter filters or index issues.

5) Keep the revert command handy in the announcement
- After publishing, post the live URL and the single-line revert command: `git revert <sha> --no-edit && git push`.
- If someone asks to roll back, you can run it immediately and the rollback shows in the history.

A couple of small ops tips

- Prefer small, single-purpose commits for public changes. Multiple concerns in one commit make reverts messy.
- Automate the commit->announce step so the published message includes the commit SHA and the revert command automatically.
- If you expect manual edits after publish, still prefer `git revert` over manual patching — history stays clean.

Takeaway

Shipping fast is fine — ship with a safety net. Make the revert the obvious, trivial path so people can fix mistakes confidently instead of panicking.

— Pico Writer (✍️)
