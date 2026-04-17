# Feature: Payment Channel Selection

**Parent Capability**: [Cash Disbursement](../CAPABILITY.md)
**Parent Product**: [Onigiri — Loan Origination System](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept

---

## User Story

> As a **loan officer**, I want to select or change the disbursement payment method (Cash, PromptPay, or Bank Transfer) at the confirmation screen, so that I can switch away from cash when the branch balance is insufficient and still complete the disbursement.

---

## Job-to-be-Done

When branch cash is insufficient, the system routes the application to `ConfirmationCash` regardless of whether anything changed since approval. The officer cannot proceed with Cash — they must explicitly choose PromptPay or Bank Transfer. The confirmation screen is the single place where this channel decision is made and the disbursement can be unblocked.

---

## State Covered

| State | Role |
|-------|------|
| `ConfirmationCash` | Officer reviews payment channel; if Cash is blocked, officer must select PromptPay or Bank Transfer before the confirm button becomes available |

---

## Payment Channels

| Channel | Icon | Available When | Details Displayed |
|---------|------|----------------|-------------------|
| **Cash** | 💵 | Branch cash balance ≥ disbursement amount | Branch name, available balance, disbursement amount, surplus |
| **PromptPay** | 📱 | Always (if account on record) | PromptPay ID, registered name, registered bank |
| **Bank Transfer** | 🏦 | Always (if account on record) | Bank, masked account number, account name |

---

## Acceptance Criteria

### Channel Display on Entry

- [ ] On entry to `ConfirmationCash`, all three payment channel options are shown.
- [ ] The application's current `disbursement_channel` is pre-selected.
- [ ] If the pre-selected channel is Cash and branch cash is insufficient: the Cash option renders as **disabled/blocked** (greyed, lock icon, reason shown). The officer cannot confirm with Cash.
- [ ] PromptPay and Bank Transfer options are always selectable if account data exists on the application record.
- [ ] If no PromptPay ID is on record: PromptPay option is disabled with tooltip "No PromptPay ID on file."
- [ ] If no bank account is on record: Bank Transfer option is disabled with tooltip "No bank account on file."

### Cash Blocked State (Insufficient Balance)

- [ ] When Cash is blocked, the screen displays prominently:
  - Available cash at branch
  - Disbursement amount
  - Shortfall (disbursement − available)
  - Message: "Cash unavailable — please select PromptPay or Bank Transfer to proceed."
- [ ] The acknowledgement checkbox and Confirm button are **disabled** while Cash is selected (or pre-selected) and blocked.
- [ ] The officer cannot advance the state without selecting a non-cash channel.

### Channel Selection & Save

- [ ] Selecting a channel auto-saves `disbursement_channel` on the application record immediately.
- [ ] Channel change is recorded in the audit log: old value, new value, officer ID, timestamp.
- [ ] After switching to PromptPay or Bank Transfer: the cash block is lifted, the acknowledgement checkbox becomes active, and the confirm button becomes available (once acknowledged).
- [ ] A channel change updates the application JSON — this will appear as a diff vs. `approval_snapshot` on any future `NeedConfCash` evaluation, which is correct and expected.

### Channel-Specific Detail Panel

- [ ] **Cash selected (sufficient)**: shows branch name, available balance (green), disbursement amount, surplus.
- [ ] **Cash selected (insufficient)**: shows branch name, available balance (red), disbursement amount, shortfall (red), block message. Includes a **Refresh** button to re-check live balance.
- [ ] **PromptPay selected**: shows PromptPay ID, registered name, registered bank.
- [ ] **Bank Transfer selected**: shows bank name, masked account number, account name, branch.

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|-------------------|
| Cash blocked → officer switches to PromptPay → confirm | Block lifted. Proceeds normally. Channel saved as PromptPay. |
| Cash blocked → officer clicks Refresh → balance now sufficient | Cash option unblocks. Officer may now select Cash and proceed. |
| Cash blocked → no PromptPay, no bank account on record | All channels unavailable. Officer cannot proceed — must return to Draft to add account details. Show clear message. |
| Officer switches channel then rejects confirmation | Channel change is rolled back (application returns to Draft with original channel). |
| Channel save fails (server error) | Revert selection. Show error. Do not advance. |

---

## Out of Scope

- Adding PromptPay / bank account to the application — must be done in Smart Form (Draft)
- Branch cash replenishment — operational, outside Onigiri
- Non-cash path channel selection — governed by Disbursement Orchestration capability

---

## Dependencies

| Dependency | Type | Status |
|------------|------|--------|
| Branch Cash Availability Check | [FEATURE_branch-cash-availability-check.md](FEATURE_branch-cash-availability-check.md) | Concept |
| PromptPay / bank account fields on application record | Smart Form capability | Existing |
| Audit log for channel change | Application record | Existing pattern |
