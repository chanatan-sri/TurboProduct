# Feature: Approval Request

**Capability**: Pre-Approval — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri
**Engineering Owner**: TBD
**Status**: Concept

---

## User Story

As a Credit Officer, I want to submit a non-EasyPass pre-approval to a designated approver for review, so that I can proceed to a formal application once authority is confirmed.

## Job-to-be-Done

For campaigns where the risk level exceeds local CO authority, a higher authority must confirm the plan before the CO commits to a formal application. This keeps the approval decision at the right level without forcing the CO to submit a full application blind.

---

## Acceptance Criteria

1. Non-EasyPass pre-approval in `created` state has a Submit action available to the CO.
2. On submit, pre-approval transitions to `pending_approval`.
3. Approver can Approve or Reject the pre-approval.
4. On Approve: pre-approval transitions to `approved`. Expiry date is set (system default unless overridden by approver — see Pre-Approval Expiry Management).
5. On Reject: pre-approval transitions to `rejected`. No Draft application can be created from this pre-approval.
6. Approver may revise their own decision (e.g., Approve → Reject or Reject → Approve) as long as the pre-approval has not reached `converted`.
7. CO cannot change or override the approver's decision after submission.
8. Every decision and revision is recorded in the audit trail with actor, timestamp, and outcome.

---

## Edge Cases and Error States

| Scenario | Expected Behaviour |
|---|---|
| EasyPass pre-approval — CO attempts to submit for Approval Request | Submit action not available for EasyPass path — no Approval Request is generated. |
| Approver revises decision after initial Approve (before `converted`) | New decision recorded. Audit trail captures both decisions with timestamps. |
| Approver attempts to revise after pre-approval reaches `converted` | Revision blocked — pre-approval is terminal. |
| CO attempts to act on pre-approval in `pending_approval` state | CO has no action available while awaiting approver decision. |

---

## Dependencies

- Underwriting Workflow engine (Topology D) must support `pending_approval`, `approved`, and `rejected` states for pre-approval entities.
- Approver notification is deferred — out of scope for restructure 1.3.
