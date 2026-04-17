# CHANGELOG_015 — Payment Channel Selection + Branch Cash Availability Check

**Date:** 2026-04-16
**Layer Affected:** Capability · Feature · Artifact
**Capability:** Cash Disbursement
**Product:** Onigiri — Loan Origination System

---

## What Changed

### New Features

#### 1. `FEATURE_payment-channel-selection.md` (Created)
Loan officer can select the disbursement payment method (Cash / PromptPay / Bank Transfer) at the `ConfirmationCash` screen. When the branch cash balance is insufficient, the Cash option is rendered as disabled (greyed, lock icon). The officer must select PromptPay or Bank Transfer to unblock the Confirm button. Channel selection auto-saves to the application record and is written to the audit log. A channel change updates `disbursement_channel` in the application JSON — this will appear as a diff vs. `approval_snapshot` on any future `NeedConfCash` evaluation, which is correct and expected.

#### 2. `FEATURE_branch-cash-availability-check.md` (Created)
On entry to `NeedConfCash` with `disbursement_channel = Cash`, the system calls the Branch Management API (`GET /branches/{branch_id}/cash-balance`) to retrieve `available_cash` for the disbursement branch. If `disbursement_amount > available_cash`, the system routes to `ConfirmationCash` — even when there are zero field changes since approval. The Cash payment option is blocked server-side. Officer sees branch name, available balance (red), disbursement amount, and shortfall. A **Refresh Balance** button re-checks live balance; if balance is now sufficient, Cash option unblocks. API error or timeout is treated as insufficient (fail safe) — never bypass on API uncertainty.

---

### Updated Documents

#### 3. `CAPABILITY.md` — Feature Inventory, Decision Table 1, User Flow Diagram, NFRs, Open Questions

**Feature Inventory:** Two new rows added:
- Payment Channel Selection (Concept)
- Branch Cash Availability Check (Concept)

**Decision Table 1 — `NeedConfCash` gate redesigned as two sequential checks (AND logic):**

Prior design had a single condition (snapshot diff). Updated to two-condition AND gate:

| Check | Condition | Result |
|-------|-----------|--------|
| Check 1 — Snapshot Diff | Diff non-empty | → `ConfirmationCash` |
| Check 1 — Snapshot Diff | `approval_snapshot` null | → `ConfirmationCash` (fail safe) |
| Check 1 — Snapshot Diff | Diff empty | Pass → evaluate Check 2 |
| Check 2 — Cash Availability | `disbursement_channel` ≠ Cash | Pass (skipped) |
| Check 2 — Cash Availability | Cash AND `available_cash ≥ disbursement_amount` | Pass |
| Check 2 — Cash Availability | Cash AND `available_cash < disbursement_amount` | → `ConfirmationCash` (Cash blocked) |
| Check 2 — Cash Availability | Branch API error / timeout | → `ConfirmationCash` (fail safe) |
| Bypass | Check 1 Pass AND Check 2 Pass | → `CreateFacilityCash` |

**Key behavioural note added:** Cash insufficiency routes to `ConfirmationCash` **even when there are zero field changes since approval**. The snapshot diff path and the cash availability path are independent blocking conditions — either condition alone is sufficient to require officer confirmation.

**User Flow diagram:** Updated `NeedConfCash` node to show two sequential checks. New branch: Check 1 passes → Check 2 cash availability check → insufficient/API fail routes to `ConfirmationCash`.

**NFRs added (3):**
- Cash balance must be fetched live at each `NeedConfCash` evaluation and each Refresh. No cached balance permitted.
- `disbursement_amount > available_cash` block must be evaluated and enforced server-side — not UI logic only.
- If Branch Management API returns error or times out, treat as insufficient. Never bypass on API uncertainty.

**Open Question 7 added:** Branch Management API contract for cash balance check — endpoint, request schema, response schema (`available_cash`, `currency`, `as_of`), error codes, timeout SLA, auth mechanism. Blocking for `FEATURE_branch-cash-availability-check` to advance to Spec.

---

#### 4. `FEATURE_cash-confirmation-gate.md` — Updated to reflect two-check gate logic

- Acceptance criteria split into Check 1 (Snapshot Diff) and Check 2 (Branch Cash Availability), evaluated sequentially.
- New edge cases added: "No diff + Cash + balance insufficient → ConfirmationCash, Cash option blocked"; "Branch API fails → fail safe, officer sees unavailability message".
- New dependencies added: Branch Cash Availability Check, Payment Channel Selection.

---

#### 5. `artifacts/diagrams/UI_cash-confirmation-screen.html` — Payment channel selector + cash availability panel

HTML mockup updated with:
- **Payment method selector**: three channel cards — Cash (with availability status), PromptPay, Bank Transfer.
- **Cash panel (blocked state)**: branch name, available cash balance (red, ฿180,000), disbursement amount (฿285,000), shortfall (red, ฿105,000), block status bar, Refresh Balance button.
- **Cash panel (sufficient state)**: available cash balance (green), surplus, Refresh button.
- **Refresh toggle**: demo button cycles between ฿180,000 (insufficient) and ฿420,000 (sufficient) to simulate live balance check.
- **Block enforcement**: Confirm button is disabled while Cash is pre-selected and blocked; switching to PromptPay or Bank Transfer lifts the block; acknowledge checkbox activates.
- **PromptPay panel**: PromptPay ID 098-765-4321, registered name, registered bank (KBank).
- **Bank Transfer panel**: SCB, masked account number, account name.
- Consistent with Thai locale mock data (นายธนกร วงษ์สุวรรณ, Toyota Fortuner 2022).

---

## Decision Log

### D1 — Forced channel switch over warning-only

**Option considered:** Display a warning when cash is insufficient but allow the officer to confirm anyway with Cash.

**Rejected because:** A warning-only approach does not prevent over-disbursement. If the branch lacks sufficient physical cash, a Cash disbursement instruction to Core Banking will fail or create a cash shortfall at the branch with no automated recovery path. The system must enforce the block — not just inform.

**Decision:** Cash option is disabled (greyed + lock icon) when `disbursement_amount > available_cash`. Officer cannot confirm with Cash. No override path from the UI. Remediation requires either: (a) branch cash replenishment + Refresh, or (b) switching to PromptPay or Bank Transfer.

---

### D2 — Cash availability check is independent of snapshot diff

**Option considered:** Only run the cash availability check when a snapshot diff is already present (i.e., as a sub-condition of the diff path).

**Rejected because:** The cash availability problem can occur even when the application is completely unchanged since approval. A branch may have had sufficient cash at approval time but not at disbursement time. These are independent failure modes — conflating them would allow a cash-insufficient bypass when no field changes exist.

**Decision:** Two-condition AND gate. Check 1 (snapshot diff) and Check 2 (cash availability) are evaluated sequentially but independently. Either condition alone is sufficient to route to `ConfirmationCash`. Only when both conditions pass does bypass to `CreateFacilityCash` occur.

---

### D3 — Server-side enforcement of cash block

**Option considered:** Enforce the cash availability block at the UI layer only — rely on the front-end to disable the Cash option and the Confirm button.

**Rejected because:** UI-only blocks can be bypassed through direct API calls. A loan officer (or attacker) who calls the state transition endpoint directly, bypassing the UI, would be able to advance to `CreateFacilityCash` with a cash-insufficient application. Given that `CreateLoanDisbCash` carries a hard-block idempotency rule (never re-execute), an erroneous disbursement instruction cannot be undone automatically.

**Decision:** The `NeedConfCash` execution step enforces the block at the server layer. The gate check runs regardless of how the state transition is triggered (UI, API, operator tooling).

---

### D4 — API fail-safe: treat Branch Management API error as insufficient

**Option considered:** If the Branch Management API is unavailable, allow bypass (optimistic path — assume balance is sufficient).

**Rejected because:** An optimistic default on API failure risks over-disbursement if the branch genuinely lacks cash. The consequence is irreversible (cash leaves the branch). The conservative default — treat API failure as insufficient — costs at most one confirmation step. This asymmetry strongly favours the fail-safe.

**Decision:** Any API error, 5xx response, or timeout is treated as `available_cash = 0`. System routes to `ConfirmationCash`. Officer sees "Balance check unavailable — please select an alternative channel." Engineering alert fires.

---

## Files Modified or Created

| File | Action |
|------|--------|
| `capabilities/cash-disbursement/features/FEATURE_payment-channel-selection.md` | Created |
| `capabilities/cash-disbursement/features/FEATURE_branch-cash-availability-check.md` | Created |
| `capabilities/cash-disbursement/CAPABILITY.md` | Updated — Feature Inventory, Decision Table 1, User Flow, NFRs, OQ7 |
| `capabilities/cash-disbursement/features/FEATURE_cash-confirmation-gate.md` | Updated — two-check gate AC, new edge cases, new dependencies |
| `artifacts/diagrams/UI_cash-confirmation-screen.html` | Updated — payment selector, cash availability panel, block enforcement |
