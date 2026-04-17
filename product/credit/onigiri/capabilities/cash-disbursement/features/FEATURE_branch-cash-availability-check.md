# Feature: Branch Cash Availability Check

**Parent Capability**: [Cash Disbursement](../CAPABILITY.md)
**Parent Product**: [Onigiri — Loan Origination System](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept

---

## User Story

> As the **system**, I want to check the remaining cash balance at the disbursement branch before allowing cash disbursement to proceed, so that the system never attempts to disburse more cash than is physically available — and forces the officer to select an alternative payment channel when cash is insufficient.

---

## Job-to-be-Done

Even when no application data has changed since approval, a cash disbursement can fail if the branch does not have enough physical cash. This check catches that scenario at the `NeedConfCash` gate and routes the application to `ConfirmationCash`, where the officer is forced to switch to PromptPay or Bank Transfer. The block is enforced server-side — it cannot be bypassed through the UI.

---

## States Covered

| State | Role |
|-------|------|
| `NeedConfCash` | System checks branch cash balance when payment channel = Cash; routes to `ConfirmationCash` if insufficient |
| `ConfirmationCash` | Officer sees block reason and cash detail; must switch channel to proceed |

---

## Acceptance Criteria

### Gate Check (`NeedConfCash`)

- [ ] When the application enters `NeedConfCash` with `disbursement_channel = Cash`, the system calls the Branch Management API to retrieve `available_cash` for the disbursement branch.
- [ ] If `disbursement_amount > available_cash`: system routes to `ConfirmationCash`. The cash insufficiency reason is stored on the application record (available for display at `ConfirmationCash`).
- [ ] If `disbursement_amount ≤ available_cash`: this condition does not independently trigger `ConfirmationCash` (the snapshot diff check still applies).
- [ ] This check runs **independently of the snapshot diff** — cash insufficiency forces `ConfirmationCash` even when there are zero field changes since approval.
- [ ] The block is **server-side enforced**: the `NeedConfCash` execution step applies the check regardless of the UI state.

### Forced Channel Switch (`ConfirmationCash` — Cash Blocked)

- [ ] When cash is blocked, the Cash payment option is rendered as disabled (greyed, lock icon).
- [ ] The screen displays:
  - Branch name
  - Available cash balance (highlighted in red)
  - Disbursement amount
  - Shortfall = disbursement amount − available cash (highlighted in red)
- [ ] A **Refresh Balance** button allows the officer to re-check the live balance from the Branch Management API.
- [ ] On **Refresh**: if balance is now sufficient → Cash option unblocks; officer may select Cash and proceed.
- [ ] On **Refresh**: if balance still insufficient → block remains.
- [ ] The officer **cannot confirm** while Cash is selected and blocked — the Confirm button is disabled server-side.
- [ ] The officer must select PromptPay or Bank Transfer to unblock the Confirm button.

### Balance Refresh

- [ ] Each Refresh call fetches a live balance from the Branch Management API (no cached value used).
- [ ] The timestamp of the last balance check is displayed to the officer.
- [ ] Refresh is rate-limited to prevent abuse (maximum N calls per minute — value TBD with Operations).

---

## Business Rule Summary

| Condition | Outcome |
|-----------|---------|
| `disbursement_channel ≠ Cash` | Cash check skipped entirely |
| `disbursement_channel = Cash` AND `available_cash ≥ disbursement_amount` | No block — cash check passes |
| `disbursement_channel = Cash` AND `available_cash < disbursement_amount` | Route to `ConfirmationCash`; Cash option blocked; officer must switch channel |
| Cash blocked + officer switches to PromptPay or Bank Transfer | Block lifted; confirmation proceeds on new channel |
| Cash blocked + officer refreshes + balance now sufficient | Cash option unblocked; officer may confirm with Cash |

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|-------------------|
| Branch Management API returns error (5xx) | Fail safe: treat as insufficient → route to `ConfirmationCash`. Alert engineering. Do not silently bypass. |
| Branch Management API timeout | Fail safe: treat as insufficient. Alert fires. Officer sees "Balance check unavailable — please select an alternative channel." |
| Branch balance exactly equals disbursement amount | Sufficient (≥ condition). No block. |
| Officer switches to PromptPay, then switches back to Cash | Cash availability re-evaluated on switch back. If still insufficient, block re-applied. |
| All alternative channels unavailable (no PromptPay, no bank account) | All channels blocked. Officer cannot proceed. Must return to Draft to add account details. Error message displayed. |

---

## Out of Scope

- Physical cash replenishment workflow — operational, not in Onigiri
- Branch cash reservation (locking balance for in-progress disbursements) — future consideration; not in this spec
- Balance check for PromptPay or Bank Transfer — not applicable; these are not cash-limited

---

## Dependencies

| Dependency | Type | Status |
|------------|------|--------|
| Branch Management API: `GET /branches/{branch_id}/cash-balance` → `{ available_cash, currency, as_of }` | External API (Sensei or Branch Ops system) | **Blocking** — API contract required before Concept → Spec |
| Payment Channel Selection feature | [FEATURE_payment-channel-selection.md](FEATURE_payment-channel-selection.md) | Concept |
| Refresh rate-limit value | Operations | Open question |

---

## NFRs Escalated to Capability

- Balance check must use live data — no stale cache → [CAPABILITY.md NFRs](../CAPABILITY.md)
- Block enforced server-side (not UI-only) → [CAPABILITY.md NFRs](../CAPABILITY.md)
- API failure treated as insufficient (fail safe) → [CAPABILITY.md NFRs](../CAPABILITY.md)
