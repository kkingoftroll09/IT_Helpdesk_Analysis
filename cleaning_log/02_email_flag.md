# Issue 02 — Email Field: Blank Values and Domain Inconsistency

**Table:** `requestors`  
**Column:** `email`  
**Tier:** 1 — Straightforward  
**Status:** 🔲 Flagged (no fix applied — business decision required)

---

## What I Found

The `email` column had three distinct conditions:

| Flag | Count | % of Records |
|---|---|---|
| Blank | 22 | 35.5% |
| Unknown domain | 1 | 1.6% |
| OK (.com or .vn) | 39 | 63.0% |
| **Total** | **62** | **100%** |

Two separate problems exist in this column:
1. **Blank values** — 22 requestors have no email on record
2. **Domain inconsistency** — valid records use both `@company.com`
and `@company.vn` with no documented standard for which is correct

The 1 unknown domain flag belongs to `test@test.com` — a QA test
account never purged from production after go-live.

---

## Verifying the 35.5% Figure

Before reporting the blank rate, the assumption behind it was verified:
*does 1 row = 1 unique person?*

A `COUNTIF` on `requestor_name` revealed several repeated names
(e.g. `Pham Quang` appears 4 times). Each repeated name was
cross-validated against `department` and `floor` — all shared
different combinations, consistent with common Vietnamese name
patterns where the same name genuinely belongs to different people.

**Conclusion:** All 62 rows represent distinct individuals.
The 35.5% blank email rate is valid across 62 unique requestors.

---

## Why It Exists

**Blanks:** No mandatory field validation was enforced at data entry.
Since business rules do not explicitly require an email address,
the system accepted blank values without flagging them as errors.
Incomplete records accumulated over time without correction.

**Domain inconsistency:** No standardized email domain policy was
communicated or enforced. Employees from different departments or
onboarding periods may have been registered under different domains.

**Test account:** A QA account (`test@test.com`) created during
system testing was never removed from the production database
after go-live.

---

## Why No Fix Was Applied

Both problems require a business decision before any data modification:

- **Blanks** cannot be filled from the dataset — HR records or
direct employee verification would be needed. Guessing or
fabricating emails would introduce false data.
- **Domain inconsistency** cannot be resolved without confirming
whether both domains are legitimate or one is the official standard.
Standardizing to the wrong domain would corrupt contact records.

Modifying either without business confirmation risks making real
employee records uncontactable or unverifiable.

**Decision:** Flag all anomalies, document findings, escalate to
business/HR for confirmation before any fix is applied.

---

## How I Flagged It

Built a helper column `email_flag` using nested IF logic checking:
1. Is the cell blank?
2. Does it contain an `@` symbol?
3. Is the domain one of the two known variants?

    =IF(E2="","⚠️ BLANK",
      IF(ISERROR(FIND("@",E2)),"⚠️ INVALID FORMAT",
        IF(AND(NOT(ISNUMBER(FIND("@company.com",E2))),
               NOT(ISNUMBER(FIND("@company.vn",E2)))),
           "⚠️ UNKNOWN DOMAIN","OK")))


Verified results via pivot table on `email_flag` column.

---

## Result and Business Impact

22 out of 62 requestors (35.5%) have no email on record — more than
1 in 3 employees cannot be contacted or verified by email for ticket
follow-ups. This is a significant completeness gap regardless of
whether email is technically mandatory.

The `test@test.com` unknown domain flag will be addressed separately
under **Issue #11 — QA Test Account in Production**.

---

## Business Rule Referenced
> Email is not explicitly required per current business rules.
> However, blank values represent incomplete records that reduce
> data quality and operational capability.
> Flagged for business review and future rule clarification.
