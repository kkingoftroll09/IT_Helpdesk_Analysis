# Issue 14 — Resolved Date Earlier Than Created Date

**Table:** `tickets`  
**Columns:** `created_date`, `resolved_date`  
**Tier:** 3 — Business Logic Violation  
**Status:** 🔲 Flagged — requires agent/stakeholder verification

---

## What I Found

2 rows have `resolved_date` earlier than `created_date` —
a logical impossibility since a ticket cannot be resolved
before it is created.

Detected using:

    =IF(AND(C2<>"", B2<>""),
       IF(C2<B2, "⚠️ RESOLVED BEFORE CREATED", "OK"),
       "BLANK")

| Ticket | Created | Resolved | Difference | Priority |
|---|---|---|---|---|
| T0045 | 2025-01-23 17:31 | 2025-01-23 16:31 | -1 hour | P3 - Medium |
| T0188 | 2025-03-11 21:00 | 2025-03-11 17:00 | -4 hours | P1 - Critical |

**Note on expected vs actual count:**
3 rows were expected (T0045, T0188, T0390). Only 2 were
detected — T0390 was likely corrected as a side effect
of the DD/MM/YYYY → YYYY-MM-DD conversion in Issue #12,
which rearranged date components and may have resolved
the impossible sequence incidentally. This was not a
deliberate fix and T0390 should be verified separately.

---

## Detail Analysis

**T0045 — P3 Medium, C08 Access Request:**
Created 17:31, resolved 16:31 — exactly 1 hour difference.
Likely a single-digit transposition error: created time
should be 16:31 or resolved time should be 18:31.
Low business impact given P3 priority, but still requires
confirmation before correction.

**T0188 — P1 Critical, C07 System Outage (March outage period):**
Created 21:00, resolved 17:00 — exactly 4 hours difference.
The symmetry suggests the created and resolved timestamps
may have been accidentally swapped during data entry.
However this cannot be assumed — the ticket may have been
created at 17:00 with resolution logged retroactively at
21:00, or another scenario entirely.

---

## Why T0188 Is Particularly Serious

T0188 is a P1 Critical System Outage ticket from the
March 10-14 outage period. A -4 hour resolution time
means this ticket would appear to have been resolved
4 hours before it was even reported — making SLA
performance look artificially strong for one of the
highest priority incidents in the dataset.

In a compliance reporting context, a falsely negative
resolution time on a P1 ticket directly affects SLA
accuracy. This is not a cosmetic issue.

---

## Why Flagging Is Correct Over Guessing

Unlike format errors (trailing spaces, unit labels,
date formats) where the correct value is deterministic,
timestamp errors have no provable correct value from
the data alone. Guessing wrong would:
- Introduce a new error while destroying evidence
  of the original
- Potentially falsify a compliance record (P1 SLA)
- Remove the audit trail needed for proper investigation

**Core principle:** Timestamps with business or compliance
implications cannot be corrected by assumption — only
by verification with the original source.

---

## Required Actions

| Ticket | Contact | Note |
|---|---|---|
| T0045 | Agent A010 (active) | Ask for actual ticket open time |
| T0188 | Agent A007 (inactive) | A007 unreachable — escalate to Tier 2 team lead or check contemporaneous records (incident logs, email, chat) from March 11, 2025 |

---

## Business Rule Referenced
> `created_date` must always be earlier than
> `resolved_date` — no exceptions.
> Violations with compliance implications must be
> verified with the source before correction.
