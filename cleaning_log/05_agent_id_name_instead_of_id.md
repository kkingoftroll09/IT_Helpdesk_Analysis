# Issue 05 — Agent ID Field Contains Name Instead of ID

**Table:** `tickets`  
**Column:** `agent_id`  
**Tier:** 2 — Logical / Relational  
**Status:** ✅ Fixed

---

## What I Found

12 rows in the `tickets` table had agent names (e.g. `Ho Tuan`,
`Dang Hoa`, `Bui Tuan`) in the `agent_id` column instead of
the expected `A###` format. All other rows followed the correct
format correctly.

Agents affected across the 12 rows:
- Nguyen Ngoc (A014) — 1 row
- Ho Tuan (A011) — 2 rows
- Ho Trang (A002) — 1 row
- Dang Hoa (A005) — 3 rows
- Bui Tuan (A012) — 3 rows
- Bui Trang (A010) — 1 row
- (A099 also visible — separate orphaned FK issue, Issue #06)

---

## Why It Exists

During a manual data import, the person populating the sheet
used a column of agent names rather than agent IDs when filling
the `agent_id` field. No input validation was enforced on the
column to reject non-ID format values, so the system accepted
name strings silently.

This is a pure data entry / import error — not an encoding
or system issue.

---

## How I Detected It

**Primary method — REGEXMATCH formula:**

    =IF(REGEXMATCH(E2, "^A[0-9]{3}$"), "OK", "⚠️ INVALID FORMAT")

How it works:
- `^` — value must start here (no leading characters)
- `A` — literal letter A
- `[0-9]{3}` — exactly 3 numeric digits
- `$` — value must end here (no trailing characters)

So `A007` passes, `Ho Tuan` fails, `A99` fails (only 2 digits),
`A0071` fails (4 digits). More precise and readable than
inferring format indirectly via ISNUMBER/MID.

This is the preferred approach whenever a field has a known,
fixed format — REGEXMATCH explicitly validates the pattern
rather than working around it.

---

## How I Recovered the Correct IDs

Used INDEX/MATCH to look up each name string against the
`agents` table and retrieve the corresponding `agent_id`:

    =IFERROR(
      INDEX(agents!$A$2:$A$17,
        MATCH(E2, agents!$B$2:$B$17, 0)),
      "⚠️ NOT FOUND")

All 12 name strings returned a valid match — no `⚠️ NOT FOUND`
results, confirming every name had exactly one corresponding
agent in the agents table.

---

## Verification Before Fixing

**Checked for inactive agent matches:**
If any name string mapped to an inactive agent (A007 or A015),
the fix would require additional steps — flagging the ticket
for reassignment rather than simply correcting the ID format.

    =COUNTIF(helper_range, "A007") → 0
    =COUNTIF(helper_range, "A015") → 0

All 12 matched IDs belong to active agents — straightforward
fix with no compounding issues.

---

## How I Fixed It

- Copied the INDEX/MATCH helper column results (correct IDs)
- Pasted as **values only** (Ctrl+Shift+V) over the 12
  flagged cells in column E
- Paste as values is critical — pasting the formula itself
  would break once the helper column is deleted

**Post-fix verification:**
Re-ran the ISNUMBER/MID detection formula across all rows.
Result: 0 flagged rows remaining.

---

## Result

| Metric | Count |
|---|---|
| Rows with name instead of ID | 12 |
| Successful ID lookups | 12 |
| NOT FOUND matches | 0 |
| Inactive agent matches | 0 |
| Rows remaining after fix | 0 |

---

## Business Rule Referenced
> `agent_id` must contain an ID value in `A###` format —
> not a name string.
> All foreign key fields must reference the primary key
> format of their parent table.
