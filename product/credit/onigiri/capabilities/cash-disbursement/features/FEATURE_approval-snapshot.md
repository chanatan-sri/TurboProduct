# Feature: Approval Snapshot

**Parent Capability**: [Cash Disbursement](../CAPABILITY.md)
**Parent Product**: [Onigiri — Loan Origination System](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept

---

## User Story

> As the **system**, I want to capture a point-in-time snapshot of the full application data when the application enters the `Approval` state, so that any subsequent field changes can be detected at the `NeedConfCash` gate and surfaced to the loan officer before irreversible cash disbursement.

---

## Job-to-be-Done

Between the `Approval` state and the `NeedConfCash` gate, an application may return to `Draft` one or more times (officer requests revisions, document uploads, etc.). Fields may change during these cycles. The institution needs to know — at the moment of cash release — whether what is being disbursed still matches what the approver originally sanctioned. The snapshot is the authoritative record of what was approved, against which the current state is compared.

---

## State Covered

| State | Role |
|-------|------|
| `Approval` | Snapshot is written on entry to this state, before any officer action |

The snapshot is consumed later by [FEATURE_cash-confirmation-gate.md](FEATURE_cash-confirmation-gate.md) at the `NeedConfCash` execution step.

---

## Acceptance Criteria

### Snapshot Capture (on entry to `Approval`)

- [ ] On entry to `Approval` state, before the HWM is written and before any execution steps run, the system serializes the full current application JSON into an `approval_snapshot` field on the application record.
- [ ] The snapshot includes all application sections and fields: borrower data, guarantor data, collateral data, loan setup (amount, term, rate, disbursement channel, payment details), and document references.
- [ ] The snapshot is stored as an immutable JSON blob. Once written, it is never updated or overwritten — including on subsequent re-entries to `Approval` (e.g., if the application is referred back from underwriting). **First write wins.**
- [ ] The snapshot write and the HWM write occur in the same PG transaction. If either fails, neither is committed.
- [ ] The `approval_snapshot` field is not exposed for editing via any API or UI. It is system-managed only.

### Snapshot Availability

- [ ] The `approval_snapshot` is accessible to the `NeedConfCash` execution step at gate evaluation time.
- [ ] If the snapshot is missing or null when `NeedConfCash` evaluates (e.g., data corruption, migration gap): treat as **diff present** — route to `ConfirmationCash`. Never bypass on missing snapshot. Fail safe.

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|-------------------|
| Application enters `Approval` for the first time | Snapshot written. |
| Application is referred back to Draft, revised, and re-enters `Approval` | Snapshot already exists — **not overwritten**. First capture is authoritative. |
| Snapshot write fails (DB error) | Approval state entry is rolled back. Application stays in prior state. Error alerted. |
| `NeedConfCash` evaluates but snapshot is null | Fail safe: treat as diff present → route to `ConfirmationCash`. |
| Snapshot field is tampered with via direct DB write | Out of scope for application-layer enforcement. Audit log provides detection. |

---

## Out of Scope

- Non-cash path — the snapshot is not used for non-cash confirmation (`waiting_for_confirmation`)
- Snapshot versioning across multiple `Approval` entries — first write wins; no history of snapshots needed
- UI display of snapshot — the snapshot is an internal system record; the diff (not the raw snapshot) is shown to the officer in `ConfirmationCash`

---

## Dependencies

| Dependency | Type | Status |
|------------|------|--------|
| Application record schema: `approval_snapshot` field (immutable JSON blob) | Engineering (data model) | Requires schema change |
| `NeedConfCash` diff evaluation — consumes the snapshot | [FEATURE_cash-confirmation-gate.md](FEATURE_cash-confirmation-gate.md) | Dependent feature |

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Atomicity | Snapshot write and HWM write in single PG transaction |
| Immutability | No UPDATE path on `approval_snapshot` at application layer |
| Fail-safe | Missing snapshot always routes to confirmation — never to bypass |
