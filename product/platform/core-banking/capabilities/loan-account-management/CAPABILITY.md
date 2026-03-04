# Capability: Loan Account Management

**Product**: Core Banking — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Platform
**Product Owner**: TBD (Platform PO / Core Banking PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Maintain the authoritative lifecycle and balance state of every loan account from the moment it is created (at disbursement) to the moment it is closed (settled or written off). Every downstream system that needs to know "what is the current balance?", "what is the account status?", or "has this account ever been overdue?" queries or subscribes to Loan Account Management.

## Why It Exists (First Principles)

- **Single source of truth**: Multiple downstream systems (DaVinci, Collections, Loan Servicing, Risk Analytics) need consistent, up-to-date account state. Without one authoritative store, each system risks holding stale or inconsistent balance figures.
- **Audit trail requirement**: Financial regulations require an immutable record of every transaction against a loan account. Post-hoc reconstruction must be possible at any time.
- **Multi-product support**: Personal loans, auto loans, and SME loans share the same account lifecycle model but differ in parameters (term, currency, collateral linkage). A single, parameterised account model eliminates duplicated logic.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Account Creation | Draft | Create loan account record on `CreateFacility` command from Onigiri. Stores product type, term, currency, principal, disbursement date. |
| Account Activation | Draft | Transition account to Active state on `DisburseLoan` command. Emits `LoanDisbursed` event. |
| Balance Store | Draft | Authoritative real-time store of: outstanding principal, accrued interest, accrued fees, overdue components, total outstanding. Updated on every payment and accrual event. |
| Transaction Ledger | Draft | Append-only immutable record of every debit/credit against the account: disbursements, payments, interest accruals, fee charges, write-offs. |
| Account Status Management | Draft | Manages state transitions (Active → Overdue → Default → Settled / Written Off) based on DPD threshold events and payment events. Emits `LoanStatusChanged` on every transition. |
| Account Closure | Draft | Settles account on full balance payment. Records write-off on approval. Emits `LoanClosed` event. |

---

## Business Rules

### Account Status Transitions

| From | Event | Condition | To | Event Emitted |
|------|-------|-----------|-----|---------------|
| Active | DPD ticks to 1 | First missed payment | Overdue | `LoanStatusChanged` |
| Overdue | Full catch-up payment | DPD resets to 0 | Active | `LoanStatusChanged` |
| Overdue | DPD ticks to 90 | Default threshold crossed | Default | `LoanStatusChanged` |
| Default | Partial recovery | DPD drops below 90 | Overdue | `LoanStatusChanged` |
| Active / Overdue / Default | Full balance paid | Outstanding = 0 | Settled | `LoanClosed` |
| Default | Write-off approved | Formal write-off decision | Written Off | `LoanClosed` |

### Balance Components

| Component | Description |
|-----------|-------------|
| `outstanding_principal` | Remaining principal after payments |
| `accrued_interest` | Interest accrued since last payment (from Interest & Fee Calculation) |
| `accrued_fees` | Outstanding fees not yet paid |
| `overdue_principal` | Principal component of missed instalments |
| `overdue_interest` | Interest component of missed instalments |
| `total_outstanding` | Sum of all components |

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Transaction ledger immutability | No record may be deleted or modified once written. Corrections are new offsetting entries. |
| Balance consistency | `total_outstanding` must equal the sum of all components at all times (enforced at write time). |
| Audit trail completeness | Every state transition and balance change must be traceable to a source event (payment, accrual trigger, command). |

---

## Open Questions

- Which fields are required vs. optional at `CreateFacility` time? (e.g., is collateral linkage required for all product types?)
- What is the write-off approval workflow — does Core Banking own the approval, or does it receive a pre-approved write-off instruction from Collections?
- How are early settlement discounts (e.g., waived penalty interest) applied — does Core Banking own this rule or does it receive a pre-computed settlement figure from Loan Servicing?
