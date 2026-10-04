# Issue 11 — Boolean Field Format Inconsistency (is_vip_requestor)

**Table:** `tickets`  
**Column:** `is_vip_requestor`  
**Tier:** 1 — Straightforward  
**Status:** ✅ Fixed

---

## What I Found

The `is_vip_requestor` column contained 6 distinct values
representing only 2 logical states (true/false):

| Value | Count | Logical State |
|---|---|---|
| TRUE | 356 | True |
| FALSE | 342 | False |
| 1 | 11 | True |
| 0 | 14 | False |
| Yes | 19 | True |
| No | 9 | False |
| **Total** | **751** | |

---

## Why It Exists

No data type enforcement was applied to the column.
Multiple people or import scripts independently chose
their own boolean convention — one used TRUE/FALSE,
another used 1/0, another used Yes/No — with no
shared standard and no system validation to reject
non-conforming values.

---

## Format Decision: Why TRUE/FALSE

Three valid options existed. TRUE/FALSE was chosen
because this column is intended for use in Google
Sheets formula logic — filtering VIP tickets,
conditional formatting, IF-based categorization.
Google Sheets native boolean (TRUE/FALSE) integrates
directly with IF, AND, OR, and FILTER without
requiring conversion. 1/0 would need VALUE wrapping
in boolean contexts; Yes/No would need string
comparison in every formula.

---

## How I Fixed It

Used IF/OR to convert all variants to TRUE/FALSE:

    =IF(OR(K2="1", K2="Yes"), TRUE,
      IF(OR(K2="0", K2="No"), FALSE, K2))

Existing TRUE/FALSE values passed through unchanged.
Numeric 1/0 and string Yes/No converted to boolean.

**Post-fix verification:**
Pivot table showed exactly 2 distinct values.
TRUE: 386, FALSE: 365, Total: 751. ✅

---

## Result

| Metric | Before | After |
|---|---|---|
| Distinct boolean variants | 6 | 2 |
| Non-standard rows | 44 | 0 |

---

## Business Rule Referenced
> `is_vip_requestor` must use TRUE/FALSE boolean format
> consistently — no numeric or string variants.
