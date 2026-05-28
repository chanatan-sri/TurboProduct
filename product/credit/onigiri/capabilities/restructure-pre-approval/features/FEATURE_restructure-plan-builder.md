# Feature: Restructure Plan Builder

**Capability**: Restructure Pre-Approval — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Spec

---

## User Story

As a **sales officer**, I want to select an existing loan contract and build a restructuring payment plan to present to the customer, and optionally submit that plan to CA for pre-approval, so that I can offer the customer a credible restructuring proposal and reduce the downstream approval time if they agree.

## Job-to-be-Done

Give the sales officer a structured way to propose a restructured loan to a customer before the customer fills in any forms. The plan builder creates a concrete, CA-reviewable offer — not just a verbal estimate.

---

## Acceptance Criteria

### Contract Selection

1. The sales officer can search for an existing loan contract by contract number or customer ID via a Core Banking lookup.
2. The system returns contract details and displays a read-only contract summary:

| Field | Source |
|-------|--------|
| Contract Number | Core Banking |
| Customer Name | Core Banking |
| Product Type | Core Banking |
| Outstanding Balance | Core Banking |
| Current Monthly Instalment | Core Banking |
| Current Remaining Tenure (months) | Core Banking |
| Current Interest Rate | Core Banking |
| Contract Status | Core Banking (must be Active) |

3. Only **Active** contracts can be selected for restructuring. Closed, written-off, or already-restructured contracts are rejected with an explanatory message.
4. Only contracts with **no existing active pre-approval** can be selected. If the contract already has a pre-approval in `DRAFT`, `PENDING_CA_REVIEW`, `PENDING_REVISION`, `APPROVED`, or `REVISION_CONFIRMED` status, the system blocks selection with: "This contract already has an active pre-approval [reference]. A new plan can only be submitted after the existing one is closed (Rejected, Revision Rejected, or Expired)."
5. The contract details fetched from Core Banking are **snapshotted** to the pre-approval record at the time of creation. Subsequent changes in Core Banking do not alter the snapshot.

### Plan Construction

6. The sales officer enters the proposed restructuring terms:

| Field | Required | Notes |
|-------|----------|-------|
| Proposed New Tenure (months) | Yes | Must be greater than remaining current tenure |
| Proposed Monthly Instalment | Yes | Calculated from balance + rate + new tenure; or manually overridden |
| Proposed Interest Rate | Yes | May differ from current rate |
| Restructuring Fee | Optional | One-time fee included in proposal |
| Notes to CA | Optional | Free text, max 500 characters |

7. The system calculates and displays a **payment schedule preview**: month-by-month instalment breakdown for the proposed terms.
8. The system displays a side-by-side comparison of current terms vs. proposed terms for the sales officer to review before submission.

### Submission Path (Optional Pre-Approval)

9. After building the plan, the sales officer chooses one of two actions:
   - **Submit for CA Pre-Approval** → creates a pre-approval record with status `PENDING_CA_REVIEW` and routes it to the CA worklist modal.
   - **Proceed to Application** → creates a pre-approval record with status `DRAFT` and navigates directly to the application input page. No CA review is requested.

10. If "Proceed to Application" is chosen, the pre-approval record (`DRAFT`) is attached to the application as context but confers **no risk-reduction benefit** — the RAE only applies the pre-approval policy for `APPROVED` or `REVISION_CONFIRMED` records.

11. Only **one** pre-approval plan may exist per contract at any time (see constraint in AC #4). There are no multiple concurrent plans for the same contract.

### Application Attachment

12. When the customer proceeds to fill in the application, the pre-approval ID is carried into the application record as `pre_approval_id`.
13. The application input page displays a read-only banner showing the attached pre-approval plan's status. If status is `APPROVED` or `REVISION_CONFIRMED`, it displays: "Pre-approved plan attached — this application qualifies for expedited risk assessment."

---

## Edge Cases and Error States

| Condition | Expected Behavior |
|-----------|-------------------|
| Contract not found in Core Banking | Error message: "Contract not found. Verify the contract number." |
| Contract status is not Active | Error message: "This contract is not eligible for restructuring. Status: [status]." |
| Core Banking lookup times out | Error message with retry option; pre-approval record is not created until a successful lookup is confirmed |
| Sales officer submits plan with no changes from current terms | Warn: "Proposed terms are identical to current terms. Are you sure you want to submit?" |
| Sales officer returns to an existing DRAFT plan | Draft is editable until submitted for pre-approval or linked to an application |

---

## Out of Scope

- Generating a formal restructuring offer document for the customer (printing / PDF)
- Submitting the plan to Core Banking (Onigiri is read-only against contract data at this stage)
- Calculating credit bureau impact of the restructuring

---

## Dependencies

- Core Banking read-only contract lookup API (contract by number or customer ID)
- Pre-approval record data model (new entity in Onigiri)
