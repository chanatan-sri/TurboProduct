# Feature: Pre-Approval Status Visibility in BOS

**Capability**: Pre-Approval — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri
**Engineering Owner**: TBD
**Status**: Concept

---

## User Story

As a Credit Officer, I want to see the current status of a customer's pre-approval from the Customer List and Customer Detail screens in BOS, so that I know when an approval decision has been made without having to check manually.

## Job-to-be-Done

After submitting a non-EasyPass Approval Request, the CO has no application number to track — the pre-approval is not yet an application. Without a status indicator in BOS, the CO has no passive signal that the approver has acted. Status visibility closes this gap by surfacing the pre-approval state directly on the customer record.

---

## What the System Must Display

### Customer List

Display the pre-approval status next to the customer record when a pre-approval exists.

| Pre-Approval State | Display |
|---|---|
| `created` | Status label: `created` |
| `pending_approval` | Status label: `pending_approval` |
| `approved` | Status label: `approved` |
| `rejected` | Status label: `rejected` |
| `expired` | Status label: `expired` |
| `converted` | Status label: `converted` |
| No pre-approval | No indicator shown |

### Customer Detail — Pre-Approval Status Section

Display a Pre-Approval Status section on Customer Detail showing one field only: the current pre-approval status. No other fields.

| Field | Value |
|---|---|
| Pre-Approval Status | Current state of the pre-approval record (`created`, `pending_approval`, `approved`, `rejected`, `expired`, `converted`) |

If the customer has no pre-approval, the section is not shown.

---

## Acceptance Criteria

1. Customer List shows the pre-approval status for any customer that has a pre-approval record.
2. Status updates when the pre-approval state changes — CO does not need to manually reload to see the latest state.
3. Customer Detail shows a Pre-Approval Status section with the current state — no other fields — when a pre-approval record exists for that customer.
4. If the customer has no pre-approval, no status is shown on either screen.

---

## Edge Cases and Error States

| Scenario | Expected Behaviour |
|---|---|
| Pre-approval transitions to `approved` while CO is viewing Customer List | Status updates on next list refresh. |
| Customer has multiple pre-approval records | Show the most recent pre-approval. Subject to deduplication rule — see Open Questions in CAPABILITY.md. |

---

## Dependencies

- BOS must poll or subscribe to pre-approval state changes from Onigiri to keep the Customer List status current.
- Customer Detail must call Onigiri pre-approval API to fetch the pre-approval record for the customer on load.
