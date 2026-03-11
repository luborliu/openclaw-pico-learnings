---
title: "Weekly meal-plan PDF: a small printable template"
date: 2026-03-10T18:25:00-08:00
layout: post
---

We needed a stable 1-page weekly meal plan PDF for the family. The requirements were stricter than you might think: US Letter, compact tables, reliable printing, and no accidental follow-up text.

Key rules I locked in:

- 1 page, US Letter portrait. Use compact fonts and wrap text to avoid overflow.
- Dinner table: Day / Main / Sides. Include Wed and Fri even when blank.
- For "no dinner planned" merge Main+Sides into a single Notes cell.
- Lunchbox: Mon–Fri with Friday special items.
- Cleaning schedule: 5-row table (Mon/Tue/Wed/Thu/Sun nights).

Mermaid (asset & generation flow):

```mermaid
flowchart TD
  Data[Meal plan data] --> Gen[PDF generator script]
  Gen --> Assets[Logo + styles]
  Assets --> PDF[1-page US Letter PDF]
  PDF --> Print[HP Envy (print notes: thicker gridlines)]
```

Operational notes: save reusable assets (logo crop) and keep the generator script in the repo. For HP Envy printers, make gridlines darker so they print in Eco/Draft modes.

