# Restructure Development Guide

**Product**: Onigiri — Loan Origination System
**Loan Type**: `restructure`
**Audience**: Developer team building the restructure loan type
**Last Updated**: 2026-03-12

> This folder is a developer-facing reference for building the restructure feature within Onigiri. It does **not** replace the capability documents — it maps what already exists, what is partially defined, and what still needs to be specified before development can begin.

---

## What Is Restructure

A restructure is a loan modification offered to existing borrowers whose repayment ability has dropped. The business extends the loan tenor (always longer than the original) and adjusts the repayment plan. The existing loan contract is settled and a new contract is opened.

Restructure is a loan type (`application_type = restructure`) within the Onigiri application lifecycle — it follows the same workflow engine as other loan types but has distinct entry paths, form behaviour, and campaign configuration.

---

## Two Entry Paths

| Path | Description |
|---|---|
| **Via Pre-Approval** | CO uses the pre-approval screen in BOS to evaluate eligibility and select a plan before creating a Draft. Draft Initializer creates the Draft with `pre_approval_id` and `pre_approval_snapshot`. |
| **Direct (no pre-approval)** | CO creates a Draft directly without a prior pre-approval. Plan selection happens inside the Smart Form Finance Page. |

Both paths produce the same application record and follow the same workflow from Draft onwards.

---

## What to Read — By Area

### 1. User Flow and State Machine
| Document | Status | What It Covers |
|---|---|---|
| [underwriting-workflow/CAPABILITY.md](../../capabilities/underwriting-workflow/CAPABILITY.md) | ✅ Done | Topology A (direct restructure workflow), Topology D (pre-approval state machine), EasyPass routing, Change Detection |

> **Gap**: No consolidated single-page flow showing both pre-approval path and direct path end-to-end. See [FLOW.md](./FLOW.md) — to be written.

---

### 2. Pre-Approval Screen
| Document | Status | What It Covers |
|---|---|---|
| [pre-approval/CAPABILITY.md](../../capabilities/pre-approval/CAPABILITY.md) | ✅ Done | States, business rules, EasyPass bypass, tenor filter, expiry |
| [FEATURE_pre-approval-request-creation.md](../../capabilities/pre-approval/features/FEATURE_pre-approval-request-creation.md) | ✅ Done | Pre-conditions (DaVinci → Pre-Build → Plan Calculation), CO inputs, ACs |
| [FEATURE_draft-initializer.md](../../capabilities/pre-approval/features/FEATURE_draft-initializer.md) | ✅ Done | Draft creation from approved pre-approval, snapshot structure |
| [FEATURE_approval-request.md](../../capabilities/pre-approval/features/FEATURE_approval-request.md) | ✅ Done | Non-EasyPass approval submission |
| [FEATURE_pre-approval-status-visibility.md](../../capabilities/pre-approval/features/FEATURE_pre-approval-status-visibility.md) | ✅ Done | Status badge on Customer List and Customer Detail in BOS |
| [FEATURE_pre-approval-expiry-management.md](../../capabilities/pre-approval/features/FEATURE_pre-approval-expiry-management.md) | ✅ Done | Expiry logic for approved pre-approvals |

> **Gap**: `pre_approval_snapshot` JSON structure not formally defined. See [DATA_MODELS.md](./DATA_MODELS.md) — to be written.

---

### 3. Plan Calculation
| Document | Status | What It Covers |
|---|---|---|
| [PLAN_CALCULATION_API.md](./PLAN_CALCULATION_API.md) | ⚠️ To document | Call sequence, request/response schema, plan option structure (tenor, grace period, term of payment), recalculation triggers |

> **Note**: Plan Calculation is an existing LOS API — it is not a new build. The integration contract (how Onigiri calls it, what it returns, when to call it) needs to be documented before the dev team can integrate. See [PLAN_CALCULATION_API.md](./PLAN_CALCULATION_API.md) — to be written.

---

### 4. Smart Form (Application Form)

Capability reference: [smart-form/CAPABILITY.md](../../capabilities/smart-form/CAPABILITY.md)

| Feature | Restructure Status | Notes |
|---|---|---|
| Page/Section/Field Composer | ✅ No change needed | Restructure uses the existing composer engine — sections and fields driven by campaign Application Template |
| Auto-Prefill | ⚠️ Needs restructure implementation | Source: DaVinci (customer) + Core Banking (loan + collateral). Field list and editability per field not yet defined. See [PAGES.md](./PAGES.md). |
| Save Draft (Mid-Session Persistence) | ✅ No change needed | Same behaviour as `new_booking` — full JSON persisted on every save and transition |
| Stage Navigator | ✅ No change needed | Sequence driven by restructure campaign Application Template — no code change required |
| NCB Consent + OTP Flow | ❓ Open question | Restructure is for existing borrowers — NCB consent applicability unclear. Confirm if credit bureau re-inquiry is required for restructure. |
| Document Upload Interface | ⚠️ Needs restructure document list | Engine unchanged. Required document types for `restructure` not yet defined. See [DOCUMENT_REQUIREMENTS.md](./DOCUMENT_REQUIREMENTS.md). |
| Approver Data Aggregation | ⚠️ Needs restructure data groups configured | Engine unchanged (`application_type`-driven). Restructure-specific data groups not yet defined. |
| Finance Page | ❌ New feature to build | Plan selection, re-selection, Plan Calculation API call, tenor filter. Business rules documented in smart-form/CAPABILITY.md. Screen spec not yet written. See [PAGES.md](./PAGES.md). |

---

### 5. Campaign Eligibility Pre-Build
| Document | Status | What It Covers |
|---|---|---|
| [campaign-eligibility-pre-build/CAPABILITY.md](../../capabilities/campaign-eligibility-pre-build/CAPABILITY.md) | ✅ Done | Full pre-build engine: two-phase evaluation, outcome classification, Maximum Amount, worklist flags |

> **Note**: Pre-Build is campaign-type-agnostic — it produces outcome + Maximum Amount for any campaign type including restructure. No restructure-specific changes required to the Pre-Build engine.

---

### 6. Campaign Configuration (Restructure Campaign Type)
| Document | Status | What It Covers |
|---|---|---|
| [loan-campaign-configuration/CAPABILITY.md](../../capabilities/loan-campaign-configuration/CAPABILITY.md) | ⚠️ Partial | General campaign configuration dimensions documented; restructure campaign type not yet specified |

> **Gap**: What does a restructure campaign look like? What dimensions does it have? How does Plan Calculation connect to campaign config? See [CAMPAIGN_CONFIG.md](./CAMPAIGN_CONFIG.md) — to be written.

---

### 7. Risk Strategy (Restructure)
| Document | Status | What It Covers |
|---|---|---|
| [risk-assessment-engine/CAPABILITY.md](../../capabilities/risk-assessment-engine/CAPABILITY.md) | ⚠️ Partial | Risk engine documented; restructure-specific strategy attributes and threshold config not specified |

> **Gap**: What attributes are evaluated in a restructure risk strategy? What is the expected `risk_level` mapping for restructure outcomes? See [RISK_STRATEGY.md](./RISK_STRATEGY.md) — to be written.

---

### 8. Document Requirements and Verification
| Document | Status | What It Covers |
|---|---|---|
| [smart-form/CAPABILITY.md](../../capabilities/smart-form/CAPABILITY.md) | ⚠️ Partial | Rules for document upload exist; restructure-specific document list not defined |
| [disbursement-orchestration/CAPABILITY.md](../../capabilities/disbursement-orchestration/CAPABILITY.md) | ⚠️ Partial | Wasabi/Matcha integration documented for new_booking; restructure-specific verification behaviour unclear |

> **Gap**: What documents are required for a restructure application? Which are uploaded at pre-approval vs at Draft? Are there different Wasabi scan rules? See [DOCUMENT_REQUIREMENTS.md](./DOCUMENT_REQUIREMENTS.md) — to be written.

---

### 9. Core Banking Integration
| Document | Status | What It Covers |
|---|---|---|
| _(none)_ | ❌ Missing | How restructure interacts with Core Banking at disbursement — settlement of original contract, opening new contract |

> **Required before development**: Restructure closes the original loan and opens a new contract. The settlement amount calculation and Core Banking API contract for this are not defined anywhere. See [CORE_BANKING_INTEGRATION.md](./CORE_BANKING_INTEGRATION.md) — to be written.

---

## Files To Be Written (Gaps)

| File | Priority | What It Needs |
|---|---|---|
| [PLAN_CALCULATION_API.md](./PLAN_CALCULATION_API.md) | 🔴 High | API contract, request/response schema, plan option structure, recalculation trigger |
| [DATA_MODELS.md](./DATA_MODELS.md) | 🔴 High | `pre_approval_snapshot` structure, application record fields for restructure, `selected_plan` schema |
| [PAGES.md](./PAGES.md) | 🔴 High | Screen-by-screen: pre-approval screen, Finance Page states, prefill field editability list |
| [CAMPAIGN_CONFIG.md](./CAMPAIGN_CONFIG.md) | 🟡 Medium | Restructure campaign type dimensions, plan configuration, eligibility criteria shape |
| [DOCUMENT_REQUIREMENTS.md](./DOCUMENT_REQUIREMENTS.md) | 🟡 Medium | Required doc types per stage (pre-approval vs Draft), Wasabi rules for restructure |
| [CORE_BANKING_INTEGRATION.md](./CORE_BANKING_INTEGRATION.md) | 🟡 Medium | Settlement API, new contract creation, disbursement flow for restructure |
| [RISK_STRATEGY.md](./RISK_STRATEGY.md) | 🟠 Lower | Restructure-specific risk attributes, threshold configuration example |
| [FLOW.md](./FLOW.md) | 🟠 Lower | Consolidated end-to-end flow (both entry paths) in a single diagram |

---

## What You Do NOT Need to Build

The following are already implemented for `new_booking` and require only configuration — no code changes — to support restructure:

- Risk Assessment Engine rule evaluation
- Campaign Eligibility Pre-Build engine
- Underwriting workflow state machine
- DocumentDB JSON storage
- Wasabi document scan trigger
- Worklist flag display