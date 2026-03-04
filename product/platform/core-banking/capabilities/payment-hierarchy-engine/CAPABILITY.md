# Capability: Payment Hierarchy Engine

**Product**: Core Banking — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Platform
**Product Owner**: TBD (Platform PO / Core Banking PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Define and execute the rules that determine how an incoming payment is allocated across the outstanding components of a loan account. When a borrower pays ฿10,000 against an account that owes penalty interest, contractual interest, and principal, Core Banking must deterministically allocate that amount according to the product's configured hierarchy — and produce an auditable record of what was applied to what.

## Why It Exists (First Principles)

- **Regulatory and contractual obligation**: Loan agreements specify how payments are applied. Applying them incorrectly exposes the company to regulatory risk and borrower disputes.
- **Product differentiation**: Different loan products carry different allocation rules. Personal loans may prioritise interest before principal; restructured loans may apply principal first to reduce outstanding balance quickly. A configurable engine eliminates hardcoded product-specific logic.
- **Downstream accuracy**: Collections and Loan Servicing depend on the payment allocation breakdown to understand what components remain outstanding. A black-box "just update the balance" approach is insufficient.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Hierarchy Rule Configuration | Draft | Per-product-type configuration of allocation order across components: penalty interest, contractual interest, principal, fees. Configurable without code changes. |
| Payment Allocation Engine | Draft | Executes allocation rules against an incoming payment amount. Handles full, partial, over, and prepayments. Produces component-level breakdown. |
| Partial Payment Handling | Draft | When payment is insufficient to cover all components: allocates in configured hierarchy order until amount is exhausted. Records residual overdue by component. |
| Overpayment Handling | Draft | When payment exceeds total outstanding: applies to outstanding balance, records credit balance or triggers refund signal. |
| Prepayment Handling | Draft | Voluntary early payment that reduces future principal. Recalculates remaining schedule. |
| Payment Allocation Event | Draft | Emits `LoanPaymentReceived` event with full component breakdown after allocation is complete. Consumed by DaVinci, Loan Servicing, and analytics. |

---

## Business Rules

### Default Allocation Hierarchy (standard loan products)

| Priority | Component | Notes |
|----------|-----------|-------|
| 1 | Penalty interest | Applied first when overdue |
| 2 | Contractual interest | Accrued interest for the period |
| 3 | Overdue principal | Past-due principal instalments |
| 4 | Current principal | Current instalment principal |
| 5 | Fees | Outstanding fees last |

> **Note**: This is the default hierarchy. Each product type can override this order via configuration. Restructured accounts may carry a different hierarchy (e.g., principal-first to maximise recovery).

### Payment Type Decision Table

| Payment Amount | Relationship to Total Outstanding | Allocation Rule |
|---------------|-----------------------------------|-----------------|
| < Total outstanding | Partial | Allocate in hierarchy order until amount exhausted. Record residuals. |
| = Total outstanding | Full | Allocate fully. Clear all components. Trigger account closure check. |
| > Total outstanding | Over | Allocate fully. Record credit balance. Emit overpayment signal. |
| Scheduled instalment amount only | Standard | Apply as per hierarchy. |
| Voluntary extra payment | Prepayment | Reduce outstanding principal. Recalculate schedule. |

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Allocation determinism | Given the same payment amount and account state, the allocation engine must always produce the same breakdown. No randomness or ambiguity in the rules. |
| Atomicity | The allocation operation (payment application + balance update + event emission) must be atomic. No partial updates. |
| Component breakdown auditability | Every `LoanPaymentReceived` event must include a full breakdown of what was applied to each component. |

---

## Open Questions

- Is the hierarchy configuration managed by Core Banking admin UI, or is it part of the loan product configuration in Onigiri (Loan Campaign Configuration capability)?
- How are court-ordered payments (legal recovery) handled — do they bypass the standard hierarchy?
- What is the business rule when a restructured account's hierarchy differs mid-account from the original product type?
