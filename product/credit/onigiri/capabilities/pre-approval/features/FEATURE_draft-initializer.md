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

## Entry Points

The Draft Initializer can be reached via two routes for non-EasyPass approvals:

| Entry | When Available | Description |
|---|---|---|
| **BOS** → Customer Detail → Pre-Approval | Pre-approval in `approved` state | CO navigates to the pre-approval screen from BOS and triggers Convert to Draft. |
| **CO Worklist** | Pre-approval in `approved` state | Approved pre-approval surfaced in the CO's worklist. CO opens it directly and triggers Convert to Draft. |

EasyPass path has no manual entry point distinction — CO converts directly from `created` state (approval step bypassed). Both BOS and CO Worklist can surface EasyPass pre-approvals for conversion.

---

## Acceptance Criteria

1. Draft Initializer is triggered by CO-initiated Convert to Draft action:
   - **EasyPass campaign**: available from `created` state — approval step bypassed, no Approval Request required.
   - **Non-EasyPass campaign**: available from BOS or CO Worklist when pre-approval is in `approved` state (not expired or rejected).
2. On conversion, a new Draft application is created pre-populated with:
   - Selected campaign from the pre-approval
   - Existing loan reference
   - Customer data from the pre-approval context
3. The following fields are set on the application record:
   - `pre_approval_id` — reference to the originating pre-approval record
   - `pre_approval_snapshot` — immutable point-in-time copy of the pre-approval data at this conversion time
   - EasyPass routing at `pending_approval` is determined by the campaign type of the selected campaign — not a stored flag.
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

- Underwriting Workflow engine must accept a new Draft application with `pre_approval_id` and `pre_approval_snapshot` on the application record.
- Smart Form must accept pre-populated field values from the pre-approval context on Draft creation.
- Change detection execution step (Underwriting Workflow) depends on `pre_approval_snapshot` being stored correctly by this feature.
