# CHANGELOG_014: Confirmation Gate — Approval Snapshot Diff

**Date**: 2026-03-31
**Layer Affected**: Feature (new), Capability (updated)
**Author**: Product session — @CAPABILITY / @FEATURE mode

---

## What Changed

### New Feature

**`capabilities/cash-disbursement/features/FEATURE_approval-snapshot.md`** — created (Concept)

Defines the approval snapshot mechanism: when an application enters the `Approval` state, the full application JSON is serialized and stored as an immutable `approval_snapshot` field on the application record. First write wins — subsequent re-entries to `Approval` do not overwrite. This snapshot is the authoritative record of what the approver sanctioned.

Key design decisions:
- Immutable after first write (first entry to `Approval` is canonical)
- Written in the same PG transaction as the HWM update
- Missing snapshot fails safe: always routes to `ConfirmationCash`, never bypasses

---

### Updated Feature

**`capabilities/cash-disbursement/features/FEATURE_cash-confirmation-gate.md`** — updated

Replaced hardcoded loan-amount-variance logic with approval snapshot diff:

| Before | After |
|--------|-------|
| Compare `disbursement_amount` vs. `approved_amount` (single field) | Diff `approval_snapshot` vs. current application JSON (all fields) |
| Trigger: amount delta | Trigger: any field change |
| No before/after display | Officer sees field-level before/after table |

User story updated from "disbursed amount differs" to "any field changed since approval."

New edge cases added: snapshot null (fail safe), diff computation failure (fail safe), immutable snapshot across multiple Draft cycles.

---

### Updated Capability

**`capabilities/cash-disbursement/CAPABILITY.md`** — updated

- **Feature Inventory**: added `Approval Snapshot` row; updated `Cash Confirmation Gate` description
- **Decision Table 1**: replaced two-row amount comparison with three-row snapshot diff table (empty diff / non-empty diff / missing snapshot)
- **Open Questions**: resolved OQ1 (variance definition) and OQ2 (actor = Loan Officer)

---

## Rationale

The original gate only checked loan amount. An application can change in many other ways between approval and disbursement — loan term, collateral, borrower address, disbursement channel (though channel is HWM-locked, other fields are not). A field-level snapshot diff is the correct and complete solution: it catches all changes, not just amount, and presents them to the officer transparently.

This approach is also simpler than a configurable rule table (Option A, considered and set aside): no campaign configuration needed, no rule management UI, no authorization workflow for rule changes. The business rule is universal — any change from what was approved requires acknowledgment.

---

## Decision Log

| Decision | Rationale | Alternative Rejected |
|----------|-----------|----------------------|
| Full application JSON snapshot, not just loan terms | All fields can change between approval and disbursement; partial snapshot would miss changes | Snapshotting only financial fields — too narrow, creates blind spots |
| First write wins on snapshot (no overwrite on re-entry to Approval) | The canonical approval is the first one; subsequent re-entries are revisions to the same application cycle | Overwrite on each Approval re-entry — would erase the original approval record, making the diff meaningless |
| Fail safe on missing snapshot | Cash disbursement is irreversible; better to require unnecessary confirmation than to bypass when data is uncertain | Bypass on missing snapshot — unacceptable risk |
| Snapshot diff over configurable rule table (Option A) | Simpler, universal, requires no configuration; any field change is by definition noteworthy | Configurable rule table — adds PM configuration overhead and requires CRO approval for each rule change |

---

## Follow-Up Actions Required

- [ ] Engineering to define `approval_snapshot` schema change on application record (immutable JSON blob field)
- [ ] Engineering to implement field-level diff utility — consider excluding system-managed fields (e.g., `updated_at`, `state`, `hwm`) from the diff scope; include only business data fields
- [ ] Product Owner to confirm: which fields, if any, should be excluded from the diff (e.g., internal audit fields that change every cycle)
- [ ] Features to be advanced from `Concept` → `Spec` once schema and diff scope are confirmed
