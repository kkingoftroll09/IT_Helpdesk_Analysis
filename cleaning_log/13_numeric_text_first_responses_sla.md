# Issue 13 — Numeric Columns Stored as Text (first_responses, sla_breaches)

**Table:** `first_responses`, `sla_breaches`  
**Columns:** `response_time_hours` | `sla_target_hours`,
`actual_resolution_hours`, `breach_amount_hours`  
**Tier:** 1 — Straightforward  
**Status:** ✅ Fixed

---

## What I Found

4 numeric columns across 2 tables had values stored
as text instead of numbers — detectable by left
alignment in Google Sheets (numbers right-align
by default) and confirmed by ISNUMBER returning FALSE.

| Table | Column | Rows Affected |
|---|---|---|
| `first_responses` | `response_time_hours` | All rows |
| `sla_breaches` | `sla_target_hours` | All rows |
| `sla_breaches` | `actual_resolution_hours` | All rows |
| `sla_breaches` | `breach_amount_hours` | All rows |

---

## Why It Exists

No data type enforcement was applied to these columns
during import. Values were written as text strings
rather than numeric values — the system accepted
them without validation.

---

## Contrast With Issue #08

This issue shares the same surface appearance as
Issue #08 (`resolution_time_hours` stored as text)
but has a different underlying cause and simpler fix:

| | Issue #08 | Issue #13 |
|---|---|---|
| Stored value | `"4.5 hours"` | `"4.5"` |
| Root cause | Unit label baked in | Plain text, no label |
| Fix needed | SUBSTITUTE + VALUE | VALUE only |

Issue #08 required stripping the unit label before
conversion. Here, the values were already clean
number strings — only the data type was wrong.

---

## How I Detected It

Visual: left-aligned values in numeric columns.
Confirmed with:

    =ISNUMBER(VALUE(D2))

Returned TRUE — confirming values convert cleanly
to numbers without any text to strip first.

---

## How I Fixed It

Applied VALUE conversion directly:

    =VALUE(D2)

Applied to all affected columns in both tables.
Pasted as values only over original columns.

**Post-fix verification:**
AVERAGE on each column returned clean numeric results:

    =AVERAGE(first_responses!D2:D717)  → clean number ✅
    =AVERAGE(sla_breaches!C2:C79)      → clean number ✅

---

## Result

All 4 numeric columns now store proper numeric values.
Aggregate functions (AVERAGE, SUM, MIN, MAX) now
operate correctly on all rows.

---

## Business Rule Referenced
> All numeric columns must store numeric data types,
> not text strings.
> Left-alignment in a numeric column is a reliable
> visual signal of a text storage issue in
> Google Sheets.
