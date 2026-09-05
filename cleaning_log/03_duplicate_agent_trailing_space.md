# Issue 03 — Duplicate Agent Record and Trailing Space in Agent Name

**Table:** `agents`  
**Columns:** `agent_id`, `agent_name`, `active`  
**Tier:** 1 — Straightforward  
**Status:** ✅ Fixed (A015 retired) | 🔲 Flagged (A011 future hire date)

---

## What I Found

**Finding 1 — Trailing space in agent name:**
Running `=LEN(B2)-LEN(TRIM(B2))` across all 16 agent name rows
revealed that A015 had a trailing space — `"Pham Nam "` instead
of `"Pham Nam"`. All other rows returned 0.

**Finding 2 — Duplicate agent record:**
A015 (`"Pham Nam "`) is a duplicate of A003 (`"Pham Nam"`).
All fields match identically — same team, same hire date,
same active status — differing only in agent_id and the
invisible trailing space in the name.

**Related finding — A011 future hire date:**
A011 has a hire date of `2026-01-01`, which falls outside
and after the dataset period (Jan–Jun 2025). An agent cannot
resolve tickets before their hire date. A011 currently has
zero ticket assignments so there is no downstream impact,
but the record is flagged as a data entry error.

---

## Why It Exists

No deduplication validation was enforced at data entry.
The system treated `"Pham Nam"` and `"Pham Nam "` as two
distinct strings — technically correct behavior, but a design
gap that allows invisible character differences to create
ghost duplicate records. A human entering the name with an
accidental trailing space had no system warning to stop them.

This type of invisible duplicate is particularly dangerous
because it passes visual inspection — you cannot see a
trailing space by looking at the cell value.

---

## Verification Before Fixing

Before retiring A015, cross-table impact was assessed:

**Checked `tickets.agent_id` for A003 and A015:**

    =COUNTIF($D$2:$D$753, "A003") → 0
    =COUNTIF($D$2:$D$753, "A015") → 0

**Checked for name-string references in tickets:**

    =COUNTIF($D$2:$D$753, "Pham Nam") → 0

Both IDs have zero ticket assignments and no name-string
references. Safe to retire A015 without any downstream
reassignment or orphaned foreign key risk.

---

## How I Fixed It

**Step 1 — Remove trailing space from A015 name:**
Applied `=TRIM(B17)` in a helper cell, then pasted as
value only to replace the raw cell content with the
clean string `"Pham Nam"`.

**Step 2 — Mark A015 as inactive:**
Changed A015's `active` field from `TRUE` to `FALSE`.
Added note: `"Duplicate of A003 — retired"`.

**Why retire instead of delete:**
Deleting a primary key record is irreversible and unsafe
even when current ticket counts show zero — the ID may
have been referenced in historical exports, external
systems, or audit logs outside this dataset. Marking
inactive preserves the record for traceability while
preventing future assignments. This is standard practice
in any system with relational dependencies.

---

## Result

| Action | Detail |
|---|---|
| Trailing space removed | A015 name cleaned from `"Pham Nam "` to `"Pham Nam"` |
| A015 retired | `active` set to `FALSE`, duplicate of A003 noted |
| A011 flagged | Future hire date `2026-01-01` — data entry error, zero impact currently |
| Rows deleted | 0 — no records removed |
| IDs renumbered | 0 — primary keys preserved as-is |

---

## Business Rule Referenced
> `agent_id` must contain a valid, unique identifier.
> Inactive agents must not receive new ticket assignments.
> Primary keys are never deleted or renumbered once created.
