# Issue 04 — Inactive Agent A007 Still Assigned to Tickets

**Table:** `tickets`  
**Column:** `agent_id`  
**Tier:** 2 — Logical / Relational  
**Status:** 🔲 Flagged | ⚠️ 10 tickets require urgent business action

---

## What I Found

Agent A007 is marked `active = FALSE` in the `agents` table but
remains assigned to 46 tickets in the `tickets` table.

| Status | Count |
|---|---|
| Resolved | ~26 |
| Closed | ~10 |
| Reopened | 10 |
| **Total** | **46** |

A007 is a Tier 2 Support agent hired in 2023 — an experienced
agent, which explains why they handled a disproportionate number
of high-impact tickets, particularly System Outage (C07) cases.

**Other inactive agents checked:**
A015 was also marked inactive during Issue #03. COUNTIF confirmed
A015 has zero ticket assignments — no action required.

---

## Why It Exists

**Operational cause:**
The majority of A007's reopened tickets (5 out of 10) cluster
in the March 10–14 outage period. During a major outage, agents
typically apply quick fixes to restore service as fast as possible
rather than fully resolving the root cause. This is common under
pressure — restore first, investigate later. However A007 went
inactive shortly after the outage period, meaning the follow-up
investigation and permanent fixes were never completed.

**Process cause:**
No automated reassignment was triggered when A007 was marked
inactive. The system continued to display A007 as the responsible
agent on all historical and reopened tickets without any alert.
Proper offboarding processes should include a mandatory ticket
handover step before deactivating an agent account.

---

## Why Terminal Tickets Are Left As-Is

The 36 Resolved and Closed tickets are historical records.
A007 genuinely handled those tickets while active — reassigning
them retroactively would falsify the record of who did the work.

- **Resolved** — issue was fixed and confirmed by the requestor
- **Closed** — either confirmed resolved or determined
  non-existent after investigation

These records are preserved as-is. The agent name on a closed
ticket is an accurate historical fact, not an error.

---

## Why Reopened Tickets Require Business Action

Reopened tickets are not terminal — they represent active,
unresolved work items. With A007 inactive, no one is currently
responsible for these 10 tickets. Leaving them unactioned means:
- Requestors have open issues with no assigned owner
- SLA clocks may still be running on some
- The root cause problems from the March outage remain unaddressed

**Recommended action:** Organize a handover meeting to distribute
these 10 tickets to appropriate active Tier 2 or specialist agents
based on category and priority.

---

## The 10 Reopened Tickets Requiring Reassignment

| Ticket ID | Priority | Category | Subject | Note |
|---|---|---|---|---|
| T0383 | P1 - Critical | C07 | ERP system down | ⚠️ Highest urgency |
| T0532 | P1 - Critical | C07 | Payroll system unavailable | ⚠️ Highest urgency |
| T0354 | P2 - High | C07 | CRM unreachable for all users | Outage period |
| T0501 | P3 - Medium | C04 | No internet connection | Outage period |
| T0583 | P2 - High | C07 | File server outage | Outage period |
| T0590 | P3 - Medium | C09 | Archive system unreachable | Deprecated category |
| T0641 | P2 - High | C07 | ERP system down | |
| T0034 | P2 - High | C08 | New starter system access | |
| T0379 | P2 - High | C01 | Docking station not detected | |
| T0719 | P2 - High | C02 | Software update failing | Date format error in created_date |

---

## Priority Analysis

**P1 Critical tickets (T0383, T0532):**
Both involve business-critical systems — ERP and Payroll.
Both originated during the March 10–14 outage period, suggesting
quick fixes were applied under pressure without full root cause
resolution. With A007 inactive, these are the highest urgency
items requiring immediate reassignment to a senior Tier 2 agent.

**C07 concentration:**
7 out of 10 reopened tickets are System Outage (C07) — consistent
with A007 being the primary outage handler. Their inactivity
creates a single point of failure risk for outage response
capability in the current team.

---

## Additional Observations

- **T0719** has `created_date` in `DD/MM/YYYY` format
  (`17/03/2025`) — mixed date format issue, flagged for
  Issue #03 (formatting chaos tier)
- **T0354** references requestor R061 (`test@test.com`) —
  QA test account appearing in a real ticket, flagged for
  Issue #11
- **T0590** uses deprecated category C09 — flagged for
  Issue #09

---

## Business Rule Referenced
> Inactive agents (`active = FALSE`) must not receive new
> ticket assignments.
> Reopened tickets assigned to inactive agents must be
> escalated for immediate reassignment.
