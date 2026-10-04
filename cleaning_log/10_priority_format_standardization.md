# Issue 10 — Priority Field Format Inconsistency

**Table:** `tickets`  
**Column:** `priority`  
**Tier:** 1 — Straightforward  
**Status:** ✅ Fixed

---

## What I Found

The `priority` column contained 10 distinct values where
only 4 standard values should exist:

| Non-Standard Value | Count | Maps To |
|---|---|---|
| P3 (truncated) | 4 | P3 - Medium |
| 3 (numeric) | 3 | P3 - Medium |
| MED | 1 | P3 - Medium |
| medium | 1 | P3 - Medium |
| 🟡 (emoji) | 5 | P3 - Medium |
| **Total non-standard** | **14** | |

All 14 non-standard values mapped to P3 - Medium.
No other priority levels were affected.

**Notable: the emoji variant**
The 🟡 emoji is particularly problematic beyond just
being non-standard — emojis render differently across
systems and fonts, break pattern matching and REGEXMATCH,
and can cause encoding issues on CSV export. An emoji
in a data field is never appropriate regardless of
how intuitive it seems visually.

---

## Why It Exists

Multiple people or import scripts entered priority values
independently without a shared format convention. No
input validation was enforced to restrict entries to
the 4 standard values — the field accepted any text,
including abbreviations, lowercase variants, numeric
shorthand, and emoji.

---

## How I Detected It

Built a pivot table on the `priority` column to surface
all distinct values and their counts — immediately
revealed 10 variants where 4 were expected.

---

## How I Fixed It

Used IF/OR combination to map all non-standard variants
to their correct standard value:

    =IF(OR(H2="medium",H2="MED",H2="3",H2="🟡",H2="P3"),
       "P3 - Medium", H2)

Non-standard rows replaced with `P3 - Medium`.
All other priority values passed through unchanged.

**Post-fix verification:**
Pivot table showed exactly 4 distinct values.
P3 - Medium count increased from 281 to 295 (+14). ✅

---

## Result

| Metric | Before | After |
|---|---|---|
| Distinct priority values | 10 | 4 |
| Non-standard rows | 14 | 0 |
| P3 - Medium count | 281 | 295 |

---

## Business Rule Referenced
> Valid priority values: `P1 - Critical`, `P2 - High`,
> `P3 - Medium`, `P4 - Low` only — no abbreviations,
> numeric shorthand, or emoji variants.
