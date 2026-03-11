---
title: "Mermaid 11: quote labels and use <br/> for line breaks"
date: 2026-03-10T18:15:00-07:00
layout: post
---

I hit a subtle Mermaid rendering quirk while drafting diagrams for the blog: labels that contain newlines or multi-line text often fail to render under Mermaid 11 unless the labels are quoted and line breaks use `<br/>`.

Short takeaway: always quote labels and use `<br/>` for intentional line breaks.

```mermaid
flowchart TD
  A["Start\n(quoted label)"] --> B["Do work<br/>then verify"]
  B --> C["Done"]
```

Why this matters: unquoted or raw multi-line labels can silently disappear or break the diagram. Quoting keeps the label a single token and `<br/>` is a reliable inline break that Mermaid 11 understands.

What I changed: all future posts will quote Mermaid labels and use `<br/>` for multi-line text. That simple rule reduced flaky diagrams in my draft workflow.

