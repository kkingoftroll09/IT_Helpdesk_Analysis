# Issue 06 — Orphaned Agent ID A099 (Non-Existent in Agents Table)

**Table:** `tickets`  
**Column:** `agent_id`  
**Tier:** 2 — Referential Integrity Violation  
**Status:** 🔲 Flagged | ⚠️ 1 ticket requires urgent reassignment

---

## What I Found

10 tickets in the `tickets` table reference `agent_id = A099`,
which does not exist anywhere in the `agents` table.

| Status | Count |
|---|---|
| Resolved | 9 |
| Reopened | 1 |
| **Total** | **10** |

This is a true referential integrity violation — the foreign key
`agent_id` in `tickets` points to a primary key that does not
exist in the parent table `agents`.

---

## Why It Exists

When an agent leaves the organization, their system account is
typically deleted by IT as part of offboarding. However, no
corresponding process exists to update or reassign their
historical ticket records. The `tickets` table retains the
old `agent_id` value with no matching record to reference.

The reopened ticket (T0546) was never reassigned after the
agent left — without an automated handover process triggered
by account deletion, orphaned active tickets fall through
the cracks with no visible owner in the system.

---

## How This Differs From Issue #04 (A007)

| Aspect | A007 | A099 |
|---|---|---|
| Exists in agents table | ✅ Yes | ❌ No |
| Reason | Marked inactive | Account fully deleted |
| Historical context available | ✅ Yes | ❌ No |
| FK technically valid | ✅ Yes (logical issue) | ❌ No (integrity violation) |
| Severity | Business logic violation | Referential integrity violation |

A007 is a business logic problem — the agent exists but
shouldn't be assigned. A099 is a structural problem — the
agent reference is completely untraceable.

---

## How I Detected It

**Referential integrity check formula:**

    =IF(COUNTIF(agents!$A$2:$A$17, E2)=0, "⚠️ ORPHANED", "OK")

This checks whether each `agent_id` value in `tickets` has
a matching primary key in the `agents` table. Any row
returning `⚠️ ORPHANED` is a broken foreign key reference.

This is the standard pattern for checking referential
integrity across related tables — applicable to any FK
column in the dataset.

---

## Treatment

**9 Resolved tickets:**
A099 genuinely handled these tickets while active.
Historical records are preserved as-is — retroactive
reassignment would falsify the record of who did the work.
Flagged as orphaned FK in documentation.

**1 Reopened ticket — T0546:**

| Field | Value |
|---|---|
| Ticket ID | T0546 |
| Category | C05 — Email / Collaboration |
| Priority | P2 - High |
| Status | Reopened |
| Owning Team | Business Apps |

T0546 is an active work item with no traceable owner.
Requires immediate reassignment to an active Business Apps
team agent. SLA clock may still be running.

---

## Result

| Action | Detail |
|---|---|
| Orphaned FK tickets flagged | 10 |
| Terminal tickets (no action) | 9 |
| Reopened tickets for reassignment | 1 (T0546) |
| Rows deleted | 0 |
| Agent records modified | 0 |

---

## Business Rule Referenced
> `agent_id` must exist in the `agents` table — no orphaned IDs.
> Active work items (Reopened status) must have a traceable,
> active agent assigned at all times.
