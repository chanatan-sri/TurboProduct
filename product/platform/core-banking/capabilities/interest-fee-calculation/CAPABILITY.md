# Capability: Interest & Fee Calculation

**Product**: Core Banking — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Platform
**Product Owner**: TBD (Platform PO / Core Banking PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Calculate and accrue interest and fees on loan accounts according to their product configuration. Runs on a scheduled basis (daily or monthly depending on product type) and on-demand for statement generation. Produces figures that are written to the account balance store and that serve as inputs for payment allocation, statement generation, and regulatory reporting.

## Why It Exists (First Principles)

- **Revenue recognition accuracy**: Accrued interest is a recognised asset on the company's balance sheet. Incorrect or missed accruals directly affect financial reporting.
- **Regulatory compliance**: Thai financial regulations specify how interest must be calculated and disclosed to borrowers. A centralised, auditable calculation engine ensures consistency and defensibility.
- **Penalty enforcement**: Overdue accounts attract penalty interest at rates distinct from contractual interest. A consistent penalty calculation engine prevents under- or over-collection.
- **Statement production**: Loan Servicing requires accurate, period-accurate interest figures to generate correct borrower statements.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Contractual Interest Accrual | Draft | Daily accrual of interest at the contracted rate on the outstanding principal. Supports flat rate, reducing balance, and effective rate methods. |
| Penalty Interest Accrual | Draft | Daily accrual of penalty interest on overdue components when DPD > 0. Applied at the penalty rate configured per product type. |
| Fee Amortization | Draft | Amortises one-time and recurring fees (origination fee, insurance premium, etc.) across the loan term. |
| Accrual Batch Job | Draft | Scheduled daily job that triggers accrual for all active accounts. Idempotent — safe to re-run on recovery. Produces accrual completion audit log. |
| Statement Figure Production | Draft | On-demand calculation for a given accounting period. Used by Loan Servicing for statement generation. Returns principal, interest, fees, and overdue components for the period. |
| Calculation Audit Log | Draft | Immutable log of every accrual calculation: account ID, date, rate applied, basis (outstanding balance), amount accrued, method. |

---

## Business Rules

### Interest Calculation Methods

| Method | Description | Applies To |
|--------|-------------|------------|
| Reducing Balance | Interest calculated on outstanding principal each period. As principal decreases, interest decreases. | Standard instalment loans |
| Flat Rate | Interest calculated on original principal for each period. Total interest is fixed regardless of payments. | Some personal loan products |
| Effective Rate | APR-equivalent method. Used for regulatory disclosure (BOT requirements). | All products (disclosure) |

### Penalty Interest Rules

| Condition | Rule |
|-----------|------|
| DPD = 0 | No penalty interest applies. |
| DPD 1–90 | Penalty interest accrues daily on overdue instalment components at the penalty rate (configured per product type). |
| DPD > 90 (Default) | Penalty interest continues to accrue. Rate may escalate per product configuration. |
| Account Settled | Penalty interest stops accruing on settlement date. |

### Accrual Idempotency

The daily accrual job must be idempotent: if re-run for a date where accrual has already been computed, the engine detects the existing record and skips (does not double-accrue). Audit log records the skip with reason.

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Calculation precision | All monetary calculations use decimal arithmetic (not floating point). Minimum 4 decimal places before rounding. |
| Accrual completeness | No account may have a missed accrual date. Nightly completeness check compares expected vs. actual accrual records. |
| Audit trail | Every accrual record is immutable. Rate applied and calculation basis must be stored alongside the amount. |
| Reprocessability | For any account, it must be possible to fully reconstruct the accrual history from the raw transaction ledger. |

---

## Open Questions

- What is the rounding rule for daily interest amounts (banker's rounding, always round up, etc.)? This must align with regulatory disclosure requirements.
- When a restructured account changes its interest rate mid-term, is historical accrual restated or is the new rate applied prospectively only?
- Who owns the product configuration for interest rates and penalty rates — Core Banking admin, or Onigiri's Loan Campaign Configuration capability?
