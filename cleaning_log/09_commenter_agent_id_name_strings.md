# Issue 09 — commenter_agent_id Contains Name Strings Instead of IDs

**Table:** `comments`  
**Column:** `commenter_agent_id`  
**Tier:** 2 — Referential Integrity Violation  
**Status:** ✅ Partially Fixed | 🔲 10 Customer rows flagged

---

## What I Found

28 rows in the `comments` table had agent names instead of
`A###` format IDs in the `commenter_agent_id` column.

| Author Type | Invalid Format Count | Treatment |
|---|---|---|
| Agent | 18 | ✅ Fixed via INDEX/MATCH |
| Customer | 10 | 🔲 Flagged — schema decision needed |
| **Total** | **28** | |

Additionally, 1 Agent row contained `A099` — valid ID format
but orphaned FK (agent doesn't exist in agents table).
Cross-referenced to **Issue #06**.

---

## Why It Exists

**Agent rows:**
Same root cause as Issue #05 — manual import populated
agent names instead of IDs in the `commenter_agent_id`
field. No format validation prevented name strings from
being accepted.

**Customer rows:**
This is a schema design gap rather than a simple entry
error. The `commenter_agent_id` field was designed for
agent IDs only — no equivalent field exists for customer
identity. When a customer comments, whoever built the
import had no designated field for customer identity,
so they populated the agent ID field with the customer's
name as the only available option. The data ended up
in the wrong field by design, not by mistake.

---

## Schema Gap Identified

The `comments` table has no `requestor_id` column.
This means customer commenters cannot be formally
traced back to a requestor record — only their name
string exists, in a field not designed for it.

**Recommended schema addition:**
Add a `commenter_requestor_id` column (FK to
`requestors.requestor_id`) to properly track customer
commenters separately from agent commenters.

---

## How I Fixed Agent Rows

Used INDEX/MATCH with TRIM to recover correct agent IDs.
TRIM was necessary because name strings in the comments
table contained trailing whitespace not present in the
agents table — causing MATCH to fail without normalization:

    =IFERROR(
      INDEX(agents!$A$2:$A$17,
        MATCH(TRIM(F2), agents!$B$2:$B$17, 0)),
      "⚠️ NOT FOUND")

All 17 Agent name string rows returned valid matches.
Pasted recovered IDs as values only over column F.

---

## How I Flagged Customer Rows

Applied REGEXMATCH to detect invalid format, then
cross-checked with author_type to isolate Customer rows:

    =COUNTIFS($D$2:$D$1918,"Customer",
              $G$2:$G$1918,"⚠️ INVALID FORMAT")

Result: 10 Customer rows flagged.
No values modified — business decision required on
whether to blank these out or add a proper
`commenter_requestor_id` column to the schema.

---

## Result

| Metric | Count |
|---|---|
| Total invalid format rows | 28 |
| Agent rows fixed | 17 |
| Agent rows — orphaned A099 (Issue #06) | 1 |
| Customer rows flagged | 10 |
| Remaining invalid format (Agent) | 0 |

---

## Business Rules Referenced
> `commenter_agent_id` must contain an ID value in
> `A###` format for Agent rows.
> Customer comments require a separate identifier field
> — current schema does not support this.
