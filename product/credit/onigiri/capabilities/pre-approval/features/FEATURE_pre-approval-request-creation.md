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

Before the pre-approval screen opens, the system must complete these steps in sequence:

1. Fetch customer master data from DaVinci — person detail, contract detail (including original loan tenor), collateral data.
2. Call Campaign Eligibility Pre-Build with that customer context — returns eligible campaigns with EasyPass designation per campaign.
3. Call Plan Calculation API with the eligible campaigns and customer loan context — returns available plan options per campaign. Each plan option is defined by: tenor, grace period, and term of payment.

If any call fails, or if pre-built returns no eligible campaigns, or if plan calculation returns no available options, the screen must not open.

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

- Eligible campaigns returned by pre-built, each showing: campaign name, EasyPass designation.
- For each campaign: available plan options from Plan Calculation API, each showing tenor, grace period, and term of payment.
- Filters on the plan list:
  - **EasyPass filter** — All / EasyPass only / Non-EasyPass only. Applies client-side.
  - **Tenor filter** — caps the maximum tenor shown. Tenor options that are less than or equal to the customer's original loan tenor must be disabled and not selectable. A restructure must always result in a longer tenor than the original loan — if a tenor option does not meet this condition, it is not a valid restructure option.

---

## Acceptance Criteria

1. Screen opens only when all three pre-condition calls succeed and at least one plan option is available.
2. Financial data fields are pre-filled from DaVinci and editable by the CO.
3. CO must select a reason for restructure before confirming. Additional detail and document upload are optional.
4. CO can filter by EasyPass designation and by tenor, then select one plan option.
5. Tenor filter options that are ≤ the customer's original loan tenor are disabled — CO cannot select them.
6. On confirmation, a pre-approval record is created storing: selected plan option (campaign, tenor, grace period, term of payment), customer reference, financial snapshot (CO-reviewed), reason for restructure, additional detail, and supporting document references.
7. EasyPass: pre-approval is created in `created` state. Approval step is bypassed — CO converts to Draft directly without submitting an Approval Request.
8. Non-EasyPass: pre-approval is created in `created` state. CO must submit an Approval Request and wait for approver decision before converting to Draft.
9. A non-EasyPass record in `created` state persists if the CO closes without proceeding — it is not discarded.

---

## Edge Cases and Error States

| Scenario | Expected Behaviour |
|---|---|
| Master data fetch fails | Screen blocked. CO informed. No record created. |
| Pre-built call fails | Screen blocked. CO informed. No record created. |
| Pre-built returns zero eligible campaigns | Screen blocked. CO informed. No record created. |
| Plan Calculation API fails | Screen blocked. CO informed. No record created. |
| Plan Calculation returns no options with tenor > original loan tenor | Screen blocked — no valid restructure plan available. CO informed. No record created. |
| Master data partially unavailable (e.g. no collateral) | Pre-built called with available data. Missing fields shown empty on screen. |
| CO closes screen after record created (`created` state) | Record persists — CO can return to it later. |
| CO opens pre-approval for customer with existing in-flight pre-approval | Subject to deduplication rule — see Open Questions in CAPABILITY.md. |

---

## Dependencies

- DaVinci must provide person detail, contract detail (including original loan tenor), and collateral data by customer/contract reference.
- Campaign Eligibility Pre-Build must return eligible campaigns with EasyPass designation per campaign.
- Plan Calculation API must return available plan options per campaign, each with tenor, grace period, and term of payment.
- Pre-approval screen is a dedicated screen — Smart Form is not used for plan selection.
