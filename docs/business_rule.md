# Business Rules — TechViet Solutions IT Helpdesk

Defines what "correct" looks like before any cleaning begins.
Every cleaning decision references one of these rules.

---

## SLA Targets

| Priority | First Response | Resolution Target |
|---|---|---|
| P1 - Critical | 30 minutes | 4 hours |
| P2 - High | 2 hours | 8 hours |
| P3 - Medium | 4 hours | 24 hours |
| P4 - Low | 8 hours | 72 hours |

---

## Ticket Lifecycle Rules

- `created_date` must always be earlier than `resolved_date`
- A ticket cannot be `Closed` or `Resolved` without a `resolved_date`
- `resolution_time_hours` cannot be filled if status is `Open` or `In Progress`
- `customer_satisfaction` (CSAT) can only be filled after status is `Resolved` or `Closed`
- `reopened_count` must be 0 if ticket was never reopened
- A `Closed` ticket with `reopened_count > 0` must have a follow-up resolution record

---

## Priority Rules

- Valid values: `P1 - Critical`, `P2 - High`, `P3 - Medium`, `P4 - Low` only
- P1 and P2 restricted to categories `C06` (Security Incident) and `C07` (System Outage)
- Every P1 ticket unresolved after 2 hours must have an escalation record
- Priority field cannot be blank

---

## Assignment Rules

- `agent_id` must exist in the `agents` table
- `agent_id` must contain an ID value (format `A###`) — not a name string
- Assigned agent's team should match the `owning_team` of the ticket's category
- Inactive agents (`active = FALSE`) must not receive new ticket assignments
- Catch-all account A016 (`UNASSIGNED`) should hold zero tickets

---

## CSAT Rules

- Valid range: 1 to 5 only
- Cannot be filled on `Open` or `In Progress` tickets
- CSAT = 5 on tickets with `reopened_count ≥ 2` flagged for review

---

## Date and Timestamp Rules

- All dates must follow `YYYY-MM-DD HH:MM` format
- All timestamps must be in UTC+7 (Hanoi local time)
- `created_date` must fall between `2025-01-01` and `2025-06-30`
- `first_response_at` must be later than `created_date`
- Tickets created at exactly `00:00` flagged for timestamp verification

---

## Category Rules

- `category_id` must exist in the `categories` table
- No leading or trailing whitespace in `category_id`
- Deprecated categories must not appear in tickets after deprecation date

---

## Volume and Pattern Rules (observable)

- No single agent should hold more than 15% of total ticket volume
- P1 tickets should not exceed 8% of monthly total
- Reopen rate above 10% per category flags premature closing pattern
- CSAT response rate below 50% per team makes average score analytically unreliable
- Any requestor submitting 10+ tickets/month on same category triggers hardware review flag
- Ticket volume drop >30% month-over-month without known cause flags pipeline integrity issue
