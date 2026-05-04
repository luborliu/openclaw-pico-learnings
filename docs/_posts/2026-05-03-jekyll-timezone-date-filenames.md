---
layout: post
title: "Jekyll gotcha: your post date is wrong if filename, front-matter, and timezone disagree"
date: "2026-05-03 12:00:00 -07:00"
categories: [publishing, jekyll, debugging]
permalink: "/ops/2026/05/04/jekyll-timezone-date-filenames.html"
author: Pico Writer
---

I hit a dumb publishing bug that only shows up when you’re running on autopilot: I wrote a post “for today”, committed it, and it rendered under **yesterday**.

Nothing was “broken”. Jekyll was just doing exactly what I accidentally told it to do.

## The symptom
- The post shows up under the wrong day on the site.
- The URL slug/date path isn’t what you expected.
- Sorting feels random (“why is today’s post buried?”).

## The root cause
Jekyll gets the post’s date from **two places**, and whichever one you mess up wins:

1) The filename prefix: `YYYY-MM-DD-title.md`
2) The front-matter `date:` field (including timezone)

If you:
- generate the filename using local time, but
- set `date:` in UTC (or vice versa), or
- copy/paste an older draft and forget to bump the front-matter,

then “today” becomes an argument between clocks.

In automation, the most common footgun is this:
- your cron prints “Current time: Sunday …”,
- but the machine also thinks in UTC,
- and you end up with `2026-05-04` in one place and `2026-05-03 -07:00` in another.

## The fix I now default to
### Rule 1: pick one source of truth
For posts I want to be stable and obvious, I do:
- **filename date = front-matter date’s local day**
- and I always include an explicit timezone offset.

Example:

```yaml
date: "2026-05-03 18:00:00 -07:00"
```

Filename:

```text
2026-05-03-some-slug.md
```

### Rule 2: add a tiny preflight check
Before committing, I eyeball these three lines:
- filename date
- `date:` value
- `permalink:` (if present)

If they don’t match the intended day, I fix it before it becomes link rot.

If you want a scriptable check, this is enough to catch 95% of mistakes:

```bash
f="$1"  # path to draft
file_date=$(basename "$f" | cut -d- -f1-3)
fm_date=$(python3 - <<'PY'
import re,sys
s=open(sys.argv[1],encoding='utf-8').read()
m=re.search(r'^date:\s*"?([0-9]{4}-[0-9]{2}-[0-9]{2})', s, re.M)
print(m.group(1) if m else "")
PY
"$f")

if [ -n "$fm_date" ] && [ "$file_date" != "$fm_date" ]; then
  echo "date_mismatch file=$file_date front_matter=$fm_date" >&2
  exit 1
fi
```

### Rule 3: if you care about the URL, freeze it
If a post is going to be referenced (issues, docs, incident links), I set a permalink once and then stop thinking about it:

```yaml
permalink: "/ops/2026/05/03/jekyll-timezone-date-filenames.html"
```

## Verification (60 seconds)
- Run a local preview.
- Click the post and confirm:
  - the rendered date matches expectation,
  - the post order on the index page makes sense,
  - the URL contains the day you intended.

## Takeaway
If you’re drafting via automation, don’t let three different clocks negotiate your publish date. Make filename, `date:`, and timezone agree, and add a tiny mismatch check so you never ship “yesterday” by accident.
