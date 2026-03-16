# CHANGELOG_006 — Campaign Eligibility Pre-Build Capability

**Date**: 2026-03-10
**Branch**: restructure-1.3
**Layer affected**: Capability (new) · Product (modified)

---

## What Changed

### New: Campaign Eligibility Pre-Build Capability

**File**: `capabilities/campaign-eligibility-pre-build/CAPABILITY.md`

New Capability added to Onigiri — runs a batch eligibility scan across all loans in Core Banking against all published campaigns. For each loan × campaign pair, evaluates the campaign's eligibility criteria (Phase 1) and executes the campaign's Risk Strategy via the Risk Assessment Engine (Phase 2) to produce an outcome (Pass / Pass with Criteria / Deviate / Not Pass) and a Maximum Amount. Results are surfaced as flags on the shared worklist.

Key design decisions:
- **Campaign-type-agnostic engine**: Pre-Build produces only outcome + Maximum Amount. What an eligible campaign *does* is defined entirely by the campaign configuration. Adding a new campaign type requires no Pre-Build changes.
- **Two-phase evaluation**: Phase 1 is a direct eligibility gate (no Risk Engine involved). Phase 2 runs the Risk Strategy via the Risk Assessment Engine. Only loans that pass Phase 1 proceed to Phase 2.
- **Attribute-based rules only**: Pre-Build fetches loans from Core Banking and customer data from DaVinci — not Onigiri Application JSON. JMESPath is inapplicable and blocked at Campaign Builder publish time. All rules must use `source_type: attribute`.
- **Shared worklist flags**: Results appear as flags on existing worklist entries — no separate worklist. Each loan may carry multiple flags (one per campaign).
- **Idempotent re-run**: Re-running Pre-Build updates existing results for the same loan × campaign pair without creating duplicates.

**6 features at Concept status:**

| Feature | Description |
|---|---|
| Loan Evaluation Document Assembler | Assembles normalized JSON per loan from Core Banking + DaVinci; input context for Risk Assessment Engine evaluation |
| Campaign Eligibility Pre-Filter | Phase 1 — direct eligibility gate using Eligibility Criteria dimension; loans failing any criterion are immediately `Not Pass` |
| Risk Strategy Executor (Pre-Build) | Phase 2 — runs campaign Risk Strategy via Risk Assessment Engine; produces `risk_level` + `deviation_flags` |
| Outcome Classifier | Maps `risk_level` + `deviation_flags` to four-outcome classification using campaign-configured threshold ranges |
| Maximum Amount Calculator | Computes Maximum Amount from collateral value, outstanding balance, campaign Max LTV, and campaign max loan amount cap |
| Campaign Eligibility Flag Writer | Writes outcome + Maximum Amount as a flag on the shared worklist entry per loan × campaign pair |

---

### Modified: Onigiri PRODUCT.md

**File**: `PRODUCT.md`

Added Campaign Eligibility Pre-Build to the Capability Registry table.

---

## Decisions and Rationale

| Decision | Rationale |
|---|---|
| Campaign Eligibility Pre-Build is a Capability within Onigiri, not a separate product | It reads directly from Loan Campaign Configuration (eligibility criteria + risk strategy dimensions) and uses the Risk Assessment Engine. No distinct value proposition, customer boundary, or lifecycle that would warrant a separate product. |
| Pre-Build was previously an implicit dependency — now formalized | The pre-approval CAPABILITY.md (CHANGELOG_005) referenced Campaign Eligibility Pre-Build as an external dependency without a formal document. Formalizing it as a Capability ensures it has an owner, feature inventory, and business rules. |
| Phase 1 does not use the Risk Engine | Phase 1 evaluates eligibility criteria directly — these are binary pass/fail attribute checks, not risk scoring. Running the Risk Engine for Phase 1 would be incorrect: risk scoring produces a numeric level and deviation flags, not an eligibility gate. |

---

## Documents Updated This Session

- `capabilities/campaign-eligibility-pre-build/CAPABILITY.md` — created
- `PRODUCT.md` — capability registry updated (Campaign Eligibility Pre-Build added)
