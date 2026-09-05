# Issue 01 — Department Name Standardization

**Table:** `requestors`  
**Column:** `department`  
**Tier:** 1 — Straightforward  
**Status:** ✅ Fixed

---

## What I Found

The `department` column contained 16 distinct values representing 
only 8 logical departments. Examples of variants found:

- IT Department → `IT`, `it department`, `IT Dept`, `IT Department`
- HR → `HR`, `H.R.`, `Human Resources`
- Marketing → `Marketing`, `Mktg`
- Operations → `Operations`, `Ops`, `operations`
- Executive → `Executive`, `C-Suite`
- Legal → `Legal`, `Legal Dept`

No blank cells — all 62 rows had a value, but the values were 
inconsistent across rows.

---

## Why It Exists

No data validation was enforced at the input level. Multiple people 
entered department names as free text across different forms and import 
scripts without a shared reference list. Abbreviations, capitalisation 
differences, and full-name variants all accumulated over time with no 
system to catch or reject them.

This is extremely common in organisations that grow quickly without 
formalising their data entry standards early.

---

## How I Fixed It

**Step 1 — Detection**
Used a pivot table on the `department` column to surface all distinct 
values and their row counts. This immediately revealed 16 variants 
where 8 were expected.

**Step 2 — Attempted quick fix (and why it failed)**
Applied `=TRIM(PROPER(C2))` to a helper column to fix capitalisation 
and whitespace. This resolved some issues (`it department` → 
`It Department`) but failed on abbreviations like `Mktg`, `H.R.`, 
and `Ops` — because these are *meaning* problems, not *format* problems. 
No formula can know that `Mktg` means `Marketing` without being told.

**Key insight:** `TRIM` and `PROPER` fix how a value looks. 
They cannot fix what a value means. Abbreviations and alternate 
names require a lookup/mapping approach.

**Step 3 — Mapping table**
Built a two-column reference table mapping each raw variant to its 
official standard name. Used `TRIM(LOWER())` to normalize before 
lookup so capitalisation differences wouldn't cause missed matches.

Formula used:
=IFERROR(VLOOKUP(TRIM(LOWER(C2)), mapping_range, 2, FALSE), "⚠️ UNMAPPED")

The `⚠️ UNMAPPED` flag catches any variant not covered by the mapping 
table — zero unmapped results confirmed the table was complete.

**Step 4 — Verification before replacing**
Spot-checked 5 rows manually — one per problem type — before 
overwriting original data.

**Step 5 — Replace and verify**
Pasted helper column as values only into original column C. 
Deleted helper column. Ran final pivot table to confirm result.

---

## Result

| Metric | Before | After |
|---|---|---|
| Distinct department values | 16 | 8 |
| Total rows | 62 | 62 |
| Unmapped values | — | 0 |

Final pivot confirmed exactly 8 standard department names, 
Grand Total = 62. No rows lost or duplicated during the fix.

---

## Business Rule Referenced
> `department` must follow a standardized list — 
> no freeform spelling variants.
