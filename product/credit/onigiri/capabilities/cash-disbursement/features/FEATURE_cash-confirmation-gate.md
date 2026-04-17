# Feature: Cash Confirmation Gate

**Parent Capability**: [Cash Disbursement](../CAPABILITY.md)
**Parent Product**: [Onigiri — Loan Origination System](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept

---

## User Story

> As a **loan officer**, I want to be shown which fields changed since approval before cash is released, so that I can confirm the changes are intentional before irreversible funds are committed.

---

## Job-to-be-Done

Between `Approval` and the point of cash disbursement, an application may cycle back to `Draft` and be revised. The institution needs the loan officer to explicitly acknowledge any changes to the application — not just loan amount — before cash leaves. The gate presents a before/after diff of changed fields so the officer has full visibility.

---

## States Covered

| State | Role |
|-------|------|
| `NeedConfCash` | System decision node: diffs `approval_snapshot` vs. current application JSON; routes to `ConfirmationCash` or bypasses to `CreateFacilityCash` |
| `ConfirmationCash` | Human action state: loan officer reviews changed fields and confirms or rejects cash disbursement |

---

## Acceptance Criteria

### Gate Evaluation (`NeedConfCash`) — Two Sequential Checks

The gate runs two checks in order. Bypass to `CreateFacilityCash` requires **both** to pass.

**Check 1 — Snapshot Diff**

- [ ] On entry to `NeedConfCash`, the system loads `approval_snapshot` from the application record.
- [ ] The system performs a field-level diff between `approval_snapshot` and the current application JSON.
- [ ] If the diff is **non-empty** (any field changed): route to `ConfirmationCash`. Check 2 is not evaluated.
- [ ] If `approval_snapshot` is **missing or null**: treat as diff present → route to `ConfirmationCash` (fail safe).

**Check 2 — Branch Cash Availability** *(runs only when Check 1 passes)*

- [ ] If `disbursement_channel ≠ Cash`: skip — Check 2 passes automatically.
- [ ] If `disbursement_channel = Cash`: call the Branch Management API to retrieve `available_cash` for the disbursement branch.
- [ ] If `disbursement_amount > available_cash`: route to `ConfirmationCash`. Cash option is blocked at `ConfirmationCash` — officer **must** select PromptPay or Bank Transfer.
- [ ] If Branch Management API returns an error or times out: treat as insufficient → route to `ConfirmationCash` (fail safe).
- [ ] If `disbursement_amount ≤ available_cash`: Check 2 passes.

**Bypass:** Check 1 Pass AND Check 2 Pass → advance to `CreateFacilityCash`.

- [ ] The gate evaluation is system-only. No human action is required or accepted in `NeedConfCash`.
- [ ] HWM is updated to `NeedConfCash` on entry before any checks run.

### Confirmation Screen (`ConfirmationCash`)

- [ ] The confirmation screen displays the list of changed fields with **before value** (from `approval_snapshot`) and **after value** (from current application JSON) for each changed field.
- [ ] Example display:
  ```
  Confirmation Required — the following fields changed after approval:

    Field                  At Approval        Now
    ─────────────────────────────────────────────
    disbursement_amount    300,000 THB        280,000 THB
    loan_term              12 months          9 months

  [ Confirm & Disburse ]   [ Reject ]
  ```
- [ ] The loan officer sees two explicit actions: **Confirm & Disburse** and **Reject**.
- [ ] On **Confirm**: application transitions to `CreateFacilityCash`. Officer ID, confirmation timestamp, and the field diff are recorded in the audit log.
- [ ] On **Reject**: application returns to `Draft`. A rejection reason is captured (free text or structured reason code — TBD). Officer ID and timestamp are recorded.
- [ ] The disbursement channel field is **read-only** (HWM lock enforced by Smart Form lockpoint group `DISBURSEMENT_CHANNEL`).
- [ ] No fields may be edited from the confirmation screen.

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|-------------------|
| Officer rejects confirmation | Application returns to `Draft`. Fields can be revised. Diff runs fresh on next pass through `NeedConfCash`. |
| Application returns to Draft with same fields as snapshot | Diff is empty on next pass → bypass. No confirmation needed. |
| Application returns to Draft, further revised, re-enters approval path | `approval_snapshot` is immutable (first write wins). Diff always compares against original approval snapshot. |
| `approval_snapshot` is null | Fail safe: route to `ConfirmationCash`. Do not bypass. |
| Diff computation fails (system error) | Treat as diff present: route to `ConfirmationCash`. Alert engineering. |
| No diff + Cash channel + balance insufficient | Route to `ConfirmationCash`. Cash option disabled. Officer must switch to PromptPay or Bank Transfer. |
| No diff + Cash channel + Branch API fails | Fail safe: route to `ConfirmationCash`. Officer sees "Balance check unavailable — select alternative channel." |
| No diff + Cash channel + balance sufficient | Both checks pass → bypass to `CreateFacilityCash`. |
| Officer timeout — no action within SLA | Open Question 5 — escalation path undefined. Alert must fire. |
| Supervisor override of disbursement channel (post-HWM lock) | Blocked. Server-side enforcement. Remediation: cancel and submit new application. |

---

## Out of Scope

- Confirmation logic for the non-cash path — owned by [Disbursement Orchestration](../../disbursement-orchestration/CAPABILITY.md) (`waiting_for_confirmation` state)
- Snapshot capture — covered in [FEATURE_approval-snapshot.md](FEATURE_approval-snapshot.md)
- Core Banking interaction — covered in [FEATURE_cash-facility-loan-disbursement.md](FEATURE_cash-facility-loan-disbursement.md)

---

## Dependencies

| Dependency | Type | Status |
|------------|------|--------|
| `approval_snapshot` stored on application record at `Approval` state entry | [FEATURE_approval-snapshot.md](FEATURE_approval-snapshot.md) | Concept — must be built first |
| Branch Management API cash balance check | [FEATURE_branch-cash-availability-check.md](FEATURE_branch-cash-availability-check.md) | Concept — API contract required (Open Question 7) |
| Payment channel selection at `ConfirmationCash` | [FEATURE_payment-channel-selection.md](FEATURE_payment-channel-selection.md) | Concept |
| SLA for `ConfirmationCash` timeout and escalation path | NFR (Operations) | Open Question 5 |
| Smart Form lockpoint group `DISBURSEMENT_CHANNEL` enforcement | Smart Form capability | Implemented (CHANGELOG_003) |

---

## NFRs Escalated to Capability

- Confirmation timeout alerting → [CAPABILITY.md NFRs](../CAPABILITY.md)
- Audit log for officer confirm/reject + field diff snapshot → [CAPABILITY.md NFRs](../CAPABILITY.md)
