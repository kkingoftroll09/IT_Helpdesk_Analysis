# Issue 08 — Resolution Time Stored as Text With Unit Label

**Table:** `tickets`  
**Column:** `resolution_time_hours`  
**Tier:** 1 — Straightforward  
**Status:** ✅ Fixed

---

## What I Found

27 rows had resolution time stored as a text string with a
unit label appended — `"4.5 hours"` instead of the numeric
value `4.5`. All 27 followed the same consistent format.

---

## Why It Exists

No data type enforcement was applied to the
`resolution_time_hours` column — it accepts any input,
text or number. When data was exported from another system
or entered manually, the unit label was included alongside
the value. Without a schema constraint rejecting non-numeric
input, the system accepted these text strings silently.

---

## Why This Breaks Aggregate Functions

The unit label transforms a number into a completely
different data type — a text string. Aggregate functions
(AVERAGE, SUM, MIN, MAX) only operate on numeric values.
Text cells are silently excluded from calculations with
no warning or error message.

Consequence: any AVERAGE or SUM on this column before
the fix was calculated on fewer rows than actually existed
— producing an understated result with no indication
that 27 rows were excluded. An analyst relying on that
number would have no way of knowing it was wrong without
explicitly checking data types first.

---

## How I Detected It

    =IF(ISNUMBER(VALUE(J2)), "OK", "⚠️ TEXT")

`VALUE()` attempts to convert the cell to a number.
Text strings like `"4.5 hours"` fail conversion and
return an error. `ISNUMBER()` catches that error and
flags the row.

Result: 27 rows flagged as `⚠️ TEXT`.

---

## How I Fixed It

Applied SUBSTITUTE to remove the unit label, TRIM to
clean whitespace, and VALUE to convert to number:

    =IFERROR(VALUE(TRIM(SUBSTITUTE(J2," hours",""))),J2)

- `SUBSTITUTE(J2," hours","")` — removes ` hours` suffix
- `TRIM(...)` — strips any remaining whitespace
- `VALUE(...)` — converts cleaned text to numeric value
- `IFERROR(...,J2)` — preserves original if conversion fails

Pasted results as values only over column J.

**Post-fix verification:**
Re-ran detection formula → 0 `⚠️ TEXT` rows remaining.

Ran `=AVERAGE(J2:J753)` → returned `43.14 hours`.
A clean numeric result from AVERAGE confirms all 27
cells are now proper numeric values. Note: this average
will be revisited during statistical trap analysis —
placeholder values (8,760 hours) and bulk auto-closes
(0.0 hours) are both distorting this figure.

---

## Result

| Metric | Before | After |
|---|---|---|
| Text-format rows | 27 | 0 |
| AVERAGE computable | ❌ (understated) | ✅ 43.14 hours |

---

## Business Rule Referenced
> `resolution_time_hours` must be stored as a numeric value.
> Unit labels must never be embedded in data fields —
> units belong in column headers or data dictionaries,
> not in cell values.
