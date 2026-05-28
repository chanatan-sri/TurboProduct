# Feature: Application Detail View (Approver)

**Capability**: Application Approval — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Spec

---

## User Story

As a **credit approver**, I want to see the full application detail — including all borrower inputs, system-calculated fields, bureau results, and risk assessment output — filtered to what my role is authorized to see, so that I have all the information I need to make an informed, defensible credit decision.

## Job-to-be-Done

Provide a single, complete decision surface. An approver must not need to open a second system or ask a colleague for information before clicking Approve or Reject. Every relevant data point — and only data the role is trusted to see — must be on one screen.

---

## Acceptance Criteria

1. The detail view is accessible only from the Approver Worklist, via direct URL with a valid application reference, by an authenticated user whose role matches `required_approver_role`. Any other access returns HTTP 403.
2. The page is read-only — no field edits are possible from this view. Editing is only available in Draft state via the Smart Form.
3. The page renders fields in sections (see Page Structure below). Each section is visible only if the authenticated user's role meets the minimum visibility tier defined in the capability.
4. All field values rendered from system-calculated sources (NCB result, risk level, collateral valuation, DTI, LTV) are rendered as-of the timestamp they were written to the application record. The timestamp is displayed alongside each calculated value.
5. The risk assessment evaluation trace (Tier 5) renders as a collapsible tree: Strategy → Policy → Rule → Result. Each node shows the JMESPath expression evaluated, the extracted value, the comparison result, and the risk level produced.
6. The full workflow audit trail (Tier 7) displays every state transition in chronological order: state name, actor, role, timestamp, trigger reason.
7. The Approval Decision panel is rendered at the bottom of the page (see FEATURE_approval-decision.md for the decision panel spec).
8. If the application is no longer in `pending_approval` when the page is opened (e.g., another approver has already acted), the page displays a "This application is no longer pending your review" banner and disables the decision panel. It renders the current state and the decision that was taken.

---

## Page Structure

### Section 1 — Application Header *(Tier 1 — all roles)*

| Field | Source |
|-------|--------|
| Application Reference | Application record |
| Status | Current workflow state (`pending_approval`) |
| Product Type | Campaign configuration |
| Campaign Name | Campaign configuration |
| Risk Level | RAE output |
| Required Approver Role | Application record |
| Date Entered Approval | `pending_approval` entry timestamp |
| Days Pending | Calculated |
| Originating Branch | Application record |
| Originating CO | Application record (actor who submitted from Draft) |

---

### Section 2 — Borrower Information *(Tier 2 — all roles)*

All fields from the Smart Form Borrower section, rendered read-only. Includes:
- Personal details (name, date of birth, ID number, nationality)
- Contact details (phone, address)
- Employment details (employer, occupation, tenure)
- Residence type and duration

---

### Section 3 — Guarantor Information *(Tier 2 — all roles, only if guarantor present)*

All fields from the Smart Form Guarantor section, rendered read-only. Same structure as Borrower Information.

---

### Section 4 — Loan Setup *(Tier 2 — all roles)*

All fields from the Smart Form Loan Setup section. Includes:
- Requested loan amount
- Loan term (months)
- Interest rate type
- Disbursement type (cash / non-cash)
- Purpose of loan

---

### Section 5 — Financial Summary *(Tier 3 — `SALE_AREA` group_role and above)*

System-calculated and declared financial fields. Includes:

| Field | Source | Notes |
|-------|--------|-------|
| Declared Monthly Income | Smart Form | Borrower-declared |
| Declared Monthly Expenses | Smart Form | Borrower-declared |
| Net Monthly Income | Calculated: income − expenses | |
| Total Debt Obligations | Smart Form + NCB outstanding obligations | Declared + bureau-sourced |
| Debt-to-Income Ratio (DTI) | Calculated: total obligations / gross income | Displayed as percentage |
| Loan-to-Value Ratio (LTV) | Calculated: loan amount / collateral value | Only for collateral-backed loans |
| Calculation Timestamp | System timestamp of calculation | Displayed alongside each field |

---

### Section 6 — Collateral Details *(Tier 6 — `CA` group_role and above)*

| Field | Source | Notes |
|-------|--------|-------|
| Collateral Type | Smart Form | e.g., Car Title, Land Title |
| Asset Identifier | Smart Form | Registration number / title number |
| Asset Description | Smart Form | Make, model, year (for vehicles); land area (for land title) |
| Canonical Market Rate | Dashi response | Rate as of valuation request time |
| Rate Basis | Dashi response | e.g., MMPC average, dealer survey |
| Adjusted Valuation | Dashi response | After applicable adjustments |
| Valuation Timestamp | Dashi response timestamp | |
| LTV (repeated) | Calculated | For visibility in context |

---

### Section 7 — NCB Credit Bureau Result *(Tier 4 — `CA` group_role and above)*

| Field | Source | Notes |
|-------|--------|-------|
| NCB Inquiry Reference | NCB API response | |
| Credit Score | NCB response | Numeric score |
| Risk Band | NCB response | e.g., AAA / BBB / CCC |
| Outstanding Credit Facilities | NCB response | Number and total balance |
| Delinquency History | NCB response | 12-month and 36-month delinquency flags |
| Worst Bucket (last 12 months) | NCB response | Max days past due |
| Bureau Inquiry Timestamp | NCB response timestamp | |

---

### Section 8 — Risk Assessment Result *(Tier 5 — `CA` group_role and above)*

| Field | Source | Notes |
|-------|--------|-------|
| Strategy Name | RAE execution record | |
| Aggregated Risk Level | RAE output | Max across all policies |
| Deviation Flags | RAE output | List of triggered deviation flags |
| Conditional Required Documents | RAE output | Documents required due to rule matches |
| Evaluation Timestamp | RAE execution timestamp | |
| Evaluation Trace | RAE trace log | Collapsible tree: Strategy → Policy → Rule → Result |

**Evaluation trace node format:**

```
Strategy: CarTitleDefault
  Policy 1: Customer Nationality
    Rule 1.1: borrower.nationality_id = 1 → MATCH → Risk 10
  Policy 2: Customer Age
    Rule 2.2: borrower.age between 20–70 → MATCH → Risk 10
  Policy 3: Service Area
    Rule 3.2: no qualifying address → Route → Policy 4
  Policy 4: ...
Aggregate: max(10, 10, ...) = Risk 10
Deviations: []
Required Docs: []
```

---

### Section 9 — Uploaded Documents *(Tier 2 — all roles)*

List of documents uploaded in the Smart Form Document Upload stage. For each document:
- Document type
- Upload timestamp
- Wasabi early-warning scan result (if available): PASS / FLAG / PENDING
- Thumbnail / preview link (role-gated: Tier 2 minimum to view)

---

### Section 10 — Workflow Audit Trail *(Tier 7 — CA Manager and above: risk level ≥ 40)*

Chronological list of all state transitions and actions on this application:

| Column | Content |
|--------|---------|
| Timestamp | Date and time of transition |
| From State | Previous state |
| To State | New state |
| Actor | User ID and display name |
| Role | Actor's role at time of action |
| Reason / Note | Return-to-draft reason, rejection reason, or "—" |

---

## Edge Cases and Error States

| Condition | Expected Behavior |
|-----------|-------------------|
| Application no longer in `pending_approval` | Banner shown; decision panel disabled; current state displayed |
| A calculated field (NCB, RAE, Dashi) is missing | Section renders with "Data unavailable — [reason]" placeholder; other sections unaffected |
| User's role does not meet minimum tier for a section | Section is not rendered and not referenced in the page (no "access denied" hint visible — absence is the control) |
| Application detail opened with a tampered role claim | Server re-validates role against auth token; if mismatch, HTTP 403 |

---

## Out of Scope

- Field editing from the detail view
- Document re-upload or replacement from this view
- Comparison of application versions (pre- and post-return-to-draft)

---

## Dependencies

- Risk Assessment Engine must have written the evaluation trace before this view renders Section 8.
- Dashi collateral valuation must be stored on the application record (not re-fetched at render time).
- NCB result must be stored on the application record.
- Smart Form must have written all field values to the application document.
- Authentication system must provide verifiable role claim.
