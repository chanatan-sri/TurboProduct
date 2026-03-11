# CHANGELOG_007 — Smart Form: Restructure Support

**Date**: 2026-03-12
**Branch**: restructure-1.3
**Layer affected**: Capability (modified)

---

## What Changed

### Modified: Smart Form Capability

**File**: `capabilities/smart-form/CAPABILITY.md`

Extended Smart Form to formally support the `restructure` application type across all relevant business rules. Prior to this session, the capability only had explicit rules for `new_booking` and `topup`.

---

#### Feature Inventory

- **Finance Page** added as a new feature — renders campaign and plan options within Loan Setup; supports plan selection, re-selection, and Plan Calculation API recalculation on change. Behaviour differs by `application_type` (detail in Business Rules).

---

#### Business Rules Added or Updated

**Auto-Prefill Rules — `restructure`**

- Prefill source for all restructure paths: DaVinci (customer record) + Core Banking (existing loan record)
- Prefilled fields: customer identity and profile, existing loan reference, loan financial data (outstanding balance, original tenor, DPD, contract age, interest rate), collateral data (type, valuation, appraisal date)
- Applies to both restructure entry paths (via pre-approval and direct)
- Added rule: for restructure via pre-approval, `pre_approval_snapshot` carries plan/campaign data as a starting point for the Finance Page — separate from form field prefill

**Prefill Field Editability**

- Added brief section establishing that editability is determined per field per `application_type`; detail to be defined per loan type section

**Application Types — `restructure`**

- Two entry paths formalized:
  - **(1) Via pre-approval (Draft Initializer)**: Draft created from confirmed pre-approval; form fields prefilled from DaVinci + Core Banking; Finance Page pre-populates from `pre_approval_snapshot` as starting point; CO may re-select
  - **(2) Direct (no pre-approval)**: Draft created without prior pre-approval; form fields prefilled from DaVinci + Core Banking; CO selects campaign and plan on Finance Page
- Both paths support Finance Page re-selection and Plan Calculation API recalculation
- Removed incorrect "read-only" characterisation of Finance Page for restructure via pre-approval

**Finance Page Rules by Application Type** *(new section)*

| Application Type | Behaviour |
|---|---|
| `new_booking` | No Finance Page — standard Loan Setup fields only |
| `topup` | Campaign pre-selected at worklist; Finance Page renders plan details for CO confirmation; campaign cannot be switched inside Smart Form |
| `restructure` (via pre-approval) | Finance Page pre-populates from `pre_approval_snapshot`; CO may change campaign or plan option; any change triggers Plan Calculation API recalculation |
| `restructure` (direct) | Finance Page renders eligible campaigns and plan options; CO selects; any change triggers Plan Calculation API recalculation |

- Added rule: restructure Finance Page must call Plan Calculation API whenever CO changes campaign, plan option, or payment due date — CO cannot progress to Summary until recalculation succeeds
- Added rule: pre-approval plan is the default selection, not a lock — changes in Smart Form override the snapshot; `pre_approval_snapshot` is preserved for change detection

**Tenor Filter (Restructure Only)** *(new sub-section)*

- Tenor options ≤ original loan tenor are disabled on the Finance Page for all restructure paths
- Applies to both restructure entry paths
- If no valid options exist after recalculation, CO cannot progress

**Document Upload Rules by Application Type** *(new section)*

- Required document checklist is always driven by the campaign's Application Template — not hardcoded
- `restructure` uses a restructure-specific document set defined in the restructure campaign template
- Documents uploaded at the pre-approval stage are stored on the pre-approval record, not on the Draft — if the campaign template requires the same document type, CO must upload again on the application
- Wasabi early-warning scan triggered on every upload regardless of `application_type`

**NFRs**

- Added: Plan Calculation guard — restructure Finance Page must block Summary progression until a successful Plan Calculation API response confirms the selected plan

---

## Decisions and Rationale

| Decision | Rationale |
|---|---|
| Finance Page is editable for restructure via pre-approval | The pre-approval plan is a starting point, not a contract. CO may have updated information at Draft creation time (e.g. changed payment due date, customer preference). Locking the Finance Page would prevent the CO from making a valid adjustment before submission. |
| Both restructure paths share the same prefill source | DaVinci + Core Banking is the authoritative source for loan and customer data regardless of how the Draft was initiated. The pre-approval snapshot carries plan context only — it is not a substitute for live data prefill. |
| Collateral data included in restructure prefill | Collateral value and type are required inputs for the Finance Page tenor filter and Plan Calculation API. Prefilling from the authoritative source (Core Banking / DaVinci) ensures the CO starts with accurate collateral data. |
| Document Upload rules are campaign-template-driven | Hardcoding document requirements by loan type would require code changes for every new campaign. Template-driven configuration allows document requirements to vary per campaign without engineering involvement. |

---

## Documents Updated This Session

- `capabilities/smart-form/CAPABILITY.md` — Finance Page feature added; restructure Auto-Prefill, Application Types, Finance Page Rules, Document Upload Rules, Tenor Filter, NFRs updated
