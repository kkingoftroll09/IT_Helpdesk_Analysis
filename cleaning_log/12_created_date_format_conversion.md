# Issue 12 — Mixed Date Format in created_date

**Table:** `tickets`  
**Column:** `created_date`  
**Tier:** 1 — Straightforward  
**Status:** ✅ Fixed

---

## What I Found

49 rows in `created_date` used DD/MM/YYYY HH:MM format
instead of the standard YYYY-MM-DD HH:MM format used
by all other date columns in the dataset.

The `resolved_date` column had zero format errors —
only `created_date` was affected.

---

## Why It Exists

`created_date` was likely entered manually by different
people using their local date convention (DD/MM/YYYY
is the Vietnamese standard date format). `resolved_date`
appears to have been system-generated on ticket closure,
which enforced a consistent format automatically.

The asymmetry between the two columns is itself a
diagnostic clue — manual entry produces inconsistency,
system-generated values do not.

Note: this is the same mixed date format issue
encountered in the Hanoi Retail Sales capstone project —
a recurring real-world data quality pattern.

---

## Confirming the Correct Format

YYYY-MM-DD HH:MM is the correct format because:
- It matches all other date columns in the dataset
- It follows ISO 8601 international standard
- It sorts correctly as text (lexicographic = chronological)
- It is unambiguous — DD/MM/YYYY can be misread as
  MM/DD/YYYY in international contexts

---

## How I Detected It

    =IF(REGEXMATCH(B2, "^\d{2}/\d{2}/\d{4}"),
       "⚠️ DD/MM/YYYY", "OK")

49 rows flagged across the full 752-row dataset.

---

## How I Fixed It

Used LEFT, MID, RIGHT to extract and rearrange
date components into correct order:

    =IF(REGEXMATCH(B2,"^\d{2}/\d{2}/\d{4}"),
       MID(B2,7,4)&"-"&MID(B2,4,2)&"-"&LEFT(B2,2)
       &" "&RIGHT(B2,5),
       B2)

- `LEFT(B2,2)` → DD
- `MID(B2,4,2)` → MM
- `MID(B2,7,4)` → YYYY
- `RIGHT(B2,5)` → HH:MM time component

Non-flagged rows returned unchanged.
Spot-checked 5 converted rows manually before pasting.

**Post-fix verification:**
Re-ran REGEXMATCH detection → 0 flagged rows. ✅

---

## Result

| Metric | Before | After |
|---|---|---|
| DD/MM/YYYY rows | 49 | 0 |
| resolved_date affected | 0 | 0 |

---

## Business Rule Referenced
> All dates must follow YYYY-MM-DD HH:MM format.
> DD/MM/YYYY is ambiguous and non-standard for
> this dataset.
