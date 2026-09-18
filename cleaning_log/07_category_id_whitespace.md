# Issue 07 — Leading and Trailing Whitespace in category_id

**Table:** `tickets`  
**Column:** `category_id`  
**Tier:** 2 — Referential Integrity Violation  
**Status:** ✅ Fixed

---

## What I Found

26 rows in the `tickets` table had whitespace characters
embedded in the `category_id` field — invisible to the eye
but breaking all cross-table lookups silently.

| Whitespace Type | Count |
|---|---|
| Leading space (` C04`) | 10 |
| Trailing space (`C04 `) | 16 |
| **Total affected rows** | **26** |

---

## Why It Exists

Whitespace in key columns typically originates from:
- Copy-paste from another system that included surrounding spaces
- Data entry with accidental spacebar presses
- Import scripts that didn't apply TRIM before writing values

No input validation was enforced on `category_id` to reject
values containing whitespace, so the system accepted them silently.

---

## Why This Is The Most Dangerous Type of Error

This issue is visually undetectable — `" C04"` and `"C04"` look
identical in a spreadsheet cell. No formula flags it as wrong.
No conditional formatting catches it. It passes every visual
inspection.

The damage only appears when you try to JOIN or VLOOKUP across
tables — at which point the affected rows silently disappear
from your results with no warning.

**Why JOINs and VLOOKUPs fail:**

Cross-table matching uses exact string comparison —
character by character, left to right. The moment one
character differs, the entire match fails:

    " C04" vs "C04"
    ↓ ↓
    " " ≠ "C" → NO MATCH. Comparison stops immediately.


Characters 2–4 being identical is irrelevant — the first
character mismatch ends the comparison and returns no result.

**Impact by JOIN type:**

| JOIN Type | What Happens to the 26 Whitespace Rows |
|---|---|
| INNER JOIN / VLOOKUP | Row disappears entirely — silent data loss |
| LEFT JOIN | Row appears but category columns return blank |
| RIGHT JOIN | Category appears but ticket columns return blank |
| Full match check | Returns false "not found" for a valid category |

**Demonstrated in practice:**

Running VLOOKUP on a flagged row (`" C04"`) against the
categories table returned `⚠️ NO MATCH` — despite C04
existing and being valid. All 26 flagged rows produced
the same result. This means any pivot table or analysis
joining tickets to categories would silently exclude
these 26 tickets with no error message.

In a real scenario: if the 26 whitespace rows concentrated
in C07 (System Outage), outage ticket counts would appear
artificially low in every report — quietly misleading
management without triggering any alarm.

---

## How I Detected It

**Primary detection — LEN comparison:**

    =IF(LEN(D2)-LEN(TRIM(D2))>0, "⚠️ WHITESPACE", "OK")

`LEN(D2)` counts all characters including spaces.
`LEN(TRIM(D2))` counts only after whitespace is removed.
Any difference > 0 means hidden whitespace exists.

**Type breakdown — leading vs trailing:**

    =IF(LEFT(D2,1)=" ", "⚠️ LEADING", "OK")
    =IF(RIGHT(D2,1)=" ", "⚠️ TRAILING", "OK")

Result: 10 leading, 16 trailing.

---

## How I Fixed It

Applied TRIM across all category_id values in a helper column:

    =TRIM(D2)

Pasted results as values only over column D.
Deleted helper column.

**Post-fix verification:**

1. Re-ran whitespace detection formula → 0 flagged rows
2. Re-ran VLOOKUP on previously flagged rows → correct
   category returned, no more `⚠️ NO MATCH` results

---

## Result

| Metric | Before | After |
|---|---|---|
| Rows with whitespace | 26 | 0 |
| Leading space rows | 10 | 0 |
| Trailing space rows | 16 | 0 |
| VLOOKUP failures on flagged rows | 26 | 0 |

---

## Business Rule Referenced
> `category_id` must contain no leading or trailing whitespace.
> All foreign key fields must match their parent table's
> primary key format exactly — including invisible characters.
