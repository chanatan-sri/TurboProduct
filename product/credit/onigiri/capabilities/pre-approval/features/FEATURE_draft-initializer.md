# Feature: Draft Initializer

**Capability**: Pre-Approval — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri
**Engineering Owner**: TBD
**Status**: Concept

---

## User Story

As a Credit Officer, I want to convert a pre-approval into a Draft application with the plan data already filled in, so that I do not re-enter information that was already confirmed at the pre-approval stage.

## Job-to-be-Done

Pre-approval captures a confirmed plan and customer context. Converting it to Draft should carry that information forward — not require the CO to start from scratch. The same pre-approval can be reused to create multiple Drafts if previous attempts were withdrawn or rejected. The pre-approval snapshot is stored at each conversion to enable change detection at Draft submission.

---

## Acceptance Criteria

1. Draft Initializer is triggered:
   - **EasyPass**: automatically on plan selection — no explicit CO action required. Pre-approval moves directly to `converted`.
   - **Non-EasyPass**: CO-initiated Convert to Draft action, available when pre-approval is in `approved` or `converted` state (not expired).
2. On conversion, a new Draft application is created pre-populated with:
   - Selected campaign from the pre-approval
   - Existing loan reference
   - Customer data from the pre-approval context
3. The following fields are set on the application record:
   - `easypass_flag` — set from the EasyPass value returned by pre-built for this pre-approval
   - `pre_approval_id` — reference to the originating pre-approval record
   - `pre_approval_snapshot` — immutable point-in-time copy of the pre-approval data at this conversion time
4. Pre-approval transitions to `converted` state on successful Draft creation. It remains reusable — the CO can convert the same pre-approval again as long as it is still valid.
5. The Draft application enters the standard Underwriting Workflow from `draft` state.

---

## Edge Cases and Error States

| Scenario | Expected Behaviour |
|---|---|
| Non-EasyPass pre-approval in `expired` state | Convert action not available. |
| Non-EasyPass pre-approval in `rejected` or `pending_approval` state | Convert action not available. |
| Draft creation fails (downstream error) | Pre-approval state is not changed. CO informed of error. No partial application left in an inconsistent state. |
| CO changes data in Draft after converting (non-EasyPass path) | Change detection execution step in Underwriting Workflow compares Draft data against `pre_approval_snapshot` at submission — delta triggers `pending_approval` routing. |
| CO converts the same pre-approval a second time | New Draft created. New `pre_approval_snapshot` stored on the new application record. Previous Draft from the same pre-approval is unaffected. |

---

## Dependencies

- Underwriting Workflow engine must accept a new Draft application with `pre_approval_id`, `pre_approval_snapshot`, and `easypass_flag` on the application record.
- Smart Form must accept pre-populated field values from the pre-approval context on Draft creation.
- Change detection execution step (Underwriting Workflow) depends on `pre_approval_snapshot` being stored correctly by this feature.
