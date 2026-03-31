# CHANGELOG 013: Insurance Integration

**Date**: 2026-03-30
**Layer**: Capability
**Product**: Onigiri — Loan Origination System
**Author**: Claude

---

## What Changed

Added **Insurance Integration** capability to Onigiri — enabling insurance plan selection during the Smart Form Loan Setup stage with two distinct insurance flows and premium impact on loan calculations.

### New Capability: Insurance Integration

Created `capabilities/insurance-integration/CAPABILITY.md` with:
- Two insurance types: **Loan Credit Insurance** (fetched from external API) and **Voluntary/Compulsory Insurance** (reference number lookup via external API)
- **Ontop/Deduct** business logic: total premium compared against campaign budget (`max_credit_line × insurance_budget_pct`) determines whether premium is added to loan amount (Ontop) or deducted from net disbursement (Deduct)
- External Insurance System integration contract (POST /insurance/credit/plans, GET /insurance/reference/{ref_number})
- Application JSON schema for insurance data storage
- Mermaid diagrams: selection flow, API sequence, Ontop/Deduct state machine

### 5 New Features

| Feature | Description |
|---------|-------------|
| Credit Insurance Plan Retrieval | Call external API with loan context → display eligible plans → CO selects or opts out |
| External Insurance Reference Lookup | CO enters reference number → API validates → displays policy details → CO confirms. Supports multiple references |
| Insurance Premium Ontop/Deduct Calculator | Core business logic: budget = max_credit_line × insurance_budget_pct; premium ≤ budget → Ontop, premium > budget → Deduct |
| Insurance Section in Smart Form | New section in Loan Setup stage with Credit Insurance and Voluntary/Compulsory sub-sections. Fields in `LOAN_TERMS` lockpoint group |
| Insurance Data Pass-through to Plan Calculation | Include insurance premium + Ontop/Deduct method in Plan Calculation API request; trigger recalculation on insurance change |

### 6 New Backlog Items (all 💡 CONCEPT)

- Credit Insurance Plan Retrieval
- External Insurance Reference Lookup
- Insurance Premium Ontop/Deduct Calculator
- Insurance Section in Smart Form
- Insurance Data Pass-through to Plan Calculation
- Campaign Insurance Budget Configuration (cross-capability: Loan Campaign Configuration)

### Cross-Capability Updates

| Document | Change |
|----------|--------|
| Campaign Configuration CAPABILITY.md | Added `insurance_budget_pct` to Pricing dimension and Pricing Parameters table |
| Smart Form CAPABILITY.md | Added Insurance section to Loan Setup Stage-to-Section Mapping; added insurance fields to `LOAN_TERMS` lockpoint group |
| Product Type Configuration CAPABILITY.md | Added Insurance Enablement dimension (`credit_insurance`, `voluntary_insurance` boolean flags) |
| PRODUCT.md | Added Insurance Integration to capability registry; added External Insurance System to integration map (RECEIVES/SENDS + Mermaid); updated lockpoint summary table |
| BACKLOG.md | Added 6 new CONCEPT rows; updated Last Session; appended session log entry |

---

## Rationale

- Insurance premium is a core financial input that directly changes what the customer borrows (Ontop) or receives (Deduct). Without this capability, insurance must be handled outside the loan origination flow, causing manual errors and calculation mismatches.
- Two distinct acquisition flows (internal credit insurance vs. external voluntary/compulsory) require dedicated orchestration — not a simple form field addition.
- The Ontop/Deduct threshold is campaign-configurable (`insurance_budget_pct`) to support different insurance tolerance levels across loan products.

---

## Decision Log

| Decision | Chosen | Alternative | Rationale |
|----------|--------|-------------|-----------|
| Separate capability (not Smart Form extension) | Insurance Integration as standalone capability | Add insurance as Smart Form feature | Insurance involves external API calls, Ontop/Deduct business logic, and dynamic plan lists — beyond form composition scope |
| Campaign-level `insurance_budget_pct` | Configurable per campaign (default 10%) | Fixed system-wide 10% | Different loan products may have different insurance budget thresholds |
| Both insurance types coexist on single application | Yes — total premium is the sum | Mutually exclusive | Business requirement: customer may need both credit insurance and vehicle insurance |
| Credit insurance optional | CO can opt out even if enabled | Mandatory when enabled | Business flexibility — not all customers want/need credit insurance |
| Multiple voluntary/compulsory references | Allowed (multi-reference) | Single reference only | Customer may have multiple insurance policies (e.g., vehicle + health) |
| Insurance fields in `LOAN_TERMS` lockpoint group | Locked at `Approval` HWM | Separate lockpoint group | Insurance premiums affect loan amount/disbursement — same authorization scope as other loan terms |

---

## Documents Changed

### Created
- `capabilities/insurance-integration/CAPABILITY.md`
- `capabilities/insurance-integration/features/FEATURE_credit-insurance-plan-retrieval.md`
- `capabilities/insurance-integration/features/FEATURE_external-insurance-reference-lookup.md`
- `capabilities/insurance-integration/features/FEATURE_insurance-premium-ontop-deduct-calculator.md`
- `capabilities/insurance-integration/features/FEATURE_insurance-section-smart-form.md`
- `capabilities/insurance-integration/features/FEATURE_insurance-plan-calculation-passthrough.md`
- `backlog/ITEM_credit-insurance-plan-retrieval.md`
- `backlog/ITEM_external-insurance-reference-lookup.md`
- `backlog/ITEM_insurance-premium-ontop-deduct-calculator.md`
- `backlog/ITEM_insurance-section-smart-form.md`
- `backlog/ITEM_insurance-plan-calculation-passthrough.md`
- `backlog/ITEM_campaign-insurance-budget-config.md`
- `changelogs/CHANGELOG_013_insurance-integration.md`

### Updated
- `PRODUCT.md`
- `BACKLOG.md`
- `capabilities/loan-campaign-configuration/CAPABILITY.md`
- `capabilities/smart-form/CAPABILITY.md`
- `capabilities/product-type-configuration/CAPABILITY.md`
