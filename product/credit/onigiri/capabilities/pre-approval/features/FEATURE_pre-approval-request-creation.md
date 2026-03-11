# Feature: Pre-Approval Request Creation

**Capability**: Pre-Approval — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri
**Engineering Owner**: TBD
**Status**: Concept

---

## User Story

As a Credit Officer, I want to evaluate which restructure plans my customer is eligible for before creating a formal application, so that I can make a grounded offer and ensure the right approval authority handles it.

## Job-to-be-Done

The CO needs to know what is achievable and who has the authority to approve it before committing to a full application. Pre-approval gives the CO a confirmed plan and routes the work to the correct approver upfront.

---

## Pre-conditions

Before the pre-approval screen opens, the system must:
1. Fetch customer master data from DaVinci — person detail, contract detail, collateral data.
2. Call Campaign Eligibility Pre-Build with that customer context — returns eligible campaigns and EasyPass flag per campaign.

If either call fails, or if pre-built returns no eligible campaigns, the screen must not open.

---

## What the CO Must Provide

| Input | Required | Notes |
|---|---|---|
| Reason for restructure | Yes | Dropdown selection |
| Additional detail | No | Free-text supplementary explanation |
| Supporting documents | No | File upload — e.g. medical cert, income proof |
| Financial data | Yes (primary income) | Pre-filled from DaVinci; CO may correct any field before confirming |

Financial fields pre-filled from DaVinci: primary income, supplementary income, monthly debt burden, debt end date, monthly expenses, tax due date.

---

## What the System Must Display

- Eligible campaigns returned by pre-built, each showing: campaign name, min/max tenor, EasyPass indicator.
- Filters on the campaign list: by EasyPass designation and by tenor range. Filters apply client-side — no additional API call.

---

## Acceptance Criteria

1. Screen opens only when master data fetch and pre-built both succeed and at least one eligible campaign is returned.
2. Financial data fields are pre-filled from DaVinci and editable by the CO.
3. CO must select a reason for restructure before confirming. Additional detail and document upload are optional.
4. CO can filter and select one campaign from the pre-built results.
5. On confirmation, a pre-approval record is created storing: selected campaign, customer reference, financial snapshot (CO-reviewed), reason for restructure, additional detail, and supporting document references.
6. EasyPass: pre-approval auto-converts to Draft immediately on confirmation — no further CO action needed.
7. Non-EasyPass: pre-approval is created in `created` state. CO proceeds via Approval Request separately.
8. A non-EasyPass record in `created` state persists if the CO closes without proceeding — it is not discarded.

---

## Edge Cases and Error States

| Scenario | Expected Behaviour |
|---|---|
| Master data fetch fails | Screen blocked. CO informed. No record created. |
| Pre-built call fails | Screen blocked. CO informed. No record created. |
| Pre-built returns zero eligible campaigns | Screen blocked. CO informed. No record created. |
| Master data partially unavailable (e.g. no collateral) | Pre-built called with available data. Missing fields shown empty on screen. |
| CO closes screen after record created (`created` state) | Record persists — CO can return to it later. |
| CO opens pre-approval for customer with existing in-flight pre-approval | Subject to deduplication rule — see Open Questions in CAPABILITY.md. |

---

## Dependencies

- DaVinci must provide person detail, contract detail, and collateral data by customer/contract reference.
- Campaign Eligibility Pre-Build must return eligible campaigns with EasyPass flag, min tenor, and max tenor per campaign.
- Pre-approval screen is a dedicated screen — Smart Form is not used for plan selection.
