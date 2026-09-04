# IT Helpdesk Ticketing System — Data Cleaning & Analysis Project

## Overview
A structured data cleaning and analysis project built on a synthetic 
IT helpdesk ticketing dataset modeled after real-world service desk 
operations (ServiceNow / Freshdesk style).

The dataset was intentionally designed with 39 embedded data quality 
issues spanning 8 relational tables — ranging from simple formatting 
inconsistencies to silent statistical traps that corrupt analysis 
conclusions without triggering any obvious errors.

This project documents the full process: issue detection, root cause 
identification, cleaning methodology, and analytical findings.

---

## Dataset Structure

| Table | Rows | Description |
|---|---|---|
| `agents` | 16 | Support staff — name, team, hire date, active status |
| `categories` | 9 | Ticket category catalog — including one deprecated entry |
| `requestors` | 62 | Employees who raised tickets — department, email, floor |
| `tickets` | 752 | Core fact table — priority, status, dates, CSAT, resolution time |
| `comments` | ~1,917 | Conversation thread per ticket |
| `escalations` | ~164 | Tier 1 → Tier 2 handoff events with timestamps |
| `first_responses` | ~716 | First agent response time per ticket |
| `sla_breaches` | ~78 | Tickets that exceeded SLA targets |

**Relationships:** `tickets` is the central fact table. `agents`, `categories`, 
and `requestors` are dimension tables linked by foreign keys. `comments`, 
`escalations`, `first_responses`, and `sla_breaches` are event/detail tables 
linked to `tickets` via `ticket_id`.

---

## Data Quality Issues (39 total)

Issues span 6 categories of real-world data problems:

- **Formatting chaos** — mixed date formats, inconsistent priority labels, 
boolean fields with 3 different conventions
- **Referential integrity violations** — orphaned foreign keys, deprecated 
categories still in use, agent IDs containing names instead of IDs
- **Business logic violations** — resolved dates earlier than created dates, 
CSAT filled on open tickets, impossible resolution times
- **Statistical traps** — bulk auto-closes dragging MTTR down artificially, 
non-representative CSAT samples, missing data masquerading as low volume
- **Timezone inconsistency** — UTC vs UTC+7 split from a mid-year system 
migration causing negative first-response times
- **Human/process causes** — catch-all dump accounts, priority abuse, 
QA test accounts never purged from production

Full issue log with root cause analysis: [`docs/data_issues_log.md`](docs/data_issues_log.md)

---

## Business Rules

SLA targets, valid field values, assignment rules, and volume thresholds 
used as the reference standard for defining what "correct" looks like 
before any cleaning began.

Full business rules: [`docs/business_rules.md`](docs/business_rules.md)

---

## Cleaning Log

Each issue is documented individually as it was resolved:

| Issue | Table | Status |
|---|---|---|
| Department name standardization | `requestors` | ✅ Complete |

Full cleaning log: [`cleaning_log/`](cleaning_log/)

---

## Tools Used
- Google Sheets (VLOOKUP, TRIM, LOWER, COUNTIF, SUMPRODUCT, pivot tables)
- GitHub (version control and project documentation)

---

## About
**Nguyen Hoai Nam** — Master's student in Information Technology, 
Hanoi University (HANU), expected graduation 2027.

Building toward a Business/Data Analyst role with focus on data 
quality, relational data modeling, and operational analytics.
