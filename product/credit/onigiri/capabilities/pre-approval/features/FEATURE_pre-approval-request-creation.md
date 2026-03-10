# Feature: Pre-Approval Request Creation

**Capability**: Pre-Approval — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri
**Engineering Owner**: TBD
**Status**: Concept

---

## User Story

As a Credit Officer, I want to see which restructure plans my customer is eligible for when I open pre-approval, so that I can make an informed and accurate offer without committing to a formal application.

## Job-to-be-Done

The CO needs certainty before making an offer. The plan list must reflect real eligibility — not a guess — so the CO can present viable options to the customer at the branch.

---

## Pre-condition

Campaign Eligibility Pre-Build must be called with the customer context before the pre-approval screen opens. If pre-built returns no eligible campaigns, the pre-approval screen must not open — the entry point blocks access and informs the CO that this customer has no eligible restructure plans.

---

## Acceptance Criteria

1. Pre-approval screen opens only when eligible campaigns are available from pre-built.
2. The screen displays all eligible campaigns with pricing details and EasyPass indicator.
3. CO can select one campaign from the list.
4. On plan selection confirmation, a pre-approval record is created with the selected campaign and customer reference stored. Behaviour differs by EasyPass designation:
   - **EasyPass**: pre-approval auto-converts to Draft immediately — Draft Initializer pre-populates the application; pre-approval transitions directly to `converted`.
   - **Non-EasyPass**: pre-approval record created in `created` state; CO must proceed via Approval Request before conversion is available.
5. The non-EasyPass pre-approval record in `created` state persists if the CO closes the screen without proceeding — it is not discarded.

---

## Edge Cases and Error States

| Scenario | Expected Behaviour |
|---|---|
| Pre-built call fails before screen opens | Entry point blocks navigation to pre-approval. CO informed of error. No pre-approval record created. |
| CO closes screen after plan selection (state: `created`) | Record persists in `created` state — CO can return to it later. |
| CO opens pre-approval for a customer who already has an in-flight pre-approval | Behaviour subject to deduplication rule — see Open Questions in CAPABILITY.md. |

---

## Dependencies

- Calling entry point must call Campaign Eligibility Pre-Build with customer context and block access to pre-approval if no eligible campaigns are returned.
- Smart Form — Dynamic Field Options feature must be available to render the plan selection list from pre-built results.
- Campaign Eligibility Pre-Build (external) must return eligible campaigns + EasyPass flag per campaign.
