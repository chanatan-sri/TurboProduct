# CHANGELOG 007: Product Type Builder

**Date:** 2026-03-12
**Layer:** Feature
**Capability:** Product Type Configuration
**Product:** Onigiri — Loan Origination System

---

## What Changed

Added **Product Type Builder** — a new orchestration feature that ties together the 4 existing Product Type Configuration features into a cohesive multi-step PO workflow.

### New Documents
- `FEATURE_product-type-builder.md` — 5-step wizard flow with 13 acceptance criteria, 2 Mermaid workflow diagrams, and concrete Bike Title Loan example
- `ITEM_product-type-builder.md` — backlog item (status: Concept)
- `CHANGELOG_007_product-type-builder.md` — this file

### Updated Documents
- `CAPABILITY.md` — added Product Type Builder to feature inventory table
- `BACKLOG.md` — added concept row + session log entry

---

## Rationale

The 4 existing features (Collateral Section Registry, Document Type Registration, Document Requirement Declaration, Product Type Publication Authorization) each handle one step of product type assembly. However, there was no orchestration layer defining:
- The step sequence and navigation
- Cross-step validation (e.g., conditional rule fields must exist in selected section)
- Preview/simulation before submission
- Draft persistence across sessions
- Campaign Configuration integration (how ACTIVE product types flow downstream)

The Product Type Builder fills this gap as the user-facing entry point for the entire capability.

## Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Feature vs. separate capability | Feature within Product Type Configuration | Builder orchestrates existing features; doesn't introduce new business concepts |
| Campaign integration scope | AC-11 to AC-13 cover integration | Product type version pinning ensures campaigns are isolated from product type updates |
| Bike example included | Concrete 17-step example | Makes the abstract workflow tangible; serves as reference for future product types |

## Links
- [FEATURE_product-type-builder.md](../capabilities/product-type-configuration/features/FEATURE_product-type-builder.md)
- [ITEM_product-type-builder.md](../backlog/ITEM_product-type-builder.md)
- [CAPABILITY.md](../capabilities/product-type-configuration/CAPABILITY.md)
- [BACKLOG.md](../BACKLOG.md)

---

## Update: 2026-03-16 — Runtime Flow & Mock UI

### What Changed

1. **CAPABILITY.md** — added "Product Type as Application Entry Point" business rule documenting the 5-phase CO runtime flow, plus new Mermaid diagram showing Phase A through Phase E
2. **FEATURE_product-type-builder.md** — updated Bike example Phase 5 (Operations) from 4 steps to 14 steps reflecting the correct 5-phase runtime: Data Entry → Campaign Matching (with person/product limits) → Financial Details (requested amount + tenor + summary + "Create Contract") → Document Upload (conditional rules applied) → "Send to Approver" + risk assessment
3. **co-application-flow.html** — new mock UI showing the complete CO application flow with Bike Title example

### Key Architectural Insights

- Product Type is the **entry point** for application creation (Phase A) — CO selects it before any campaign
- Campaign Matching (Phase B) evaluates eligibility + person/product limits → shows matching campaigns with max loan amount
- Document upload (Phase D) happens **after** contract creation, driven by product type's document config
- Risk assessment (Phase E) is the **last step**, triggered by "Send to Approver"
- These 5 phases confirm Product Type MUST be separate from Campaign

---

## Update: 2026-03-16 — Multi-Section Application Templates & Step 0

### What Changed

1. **CAPABILITY.md** — added "Application Template: Multi-Section Composition" business rule. An application template is composed of multiple sections (standard customer sections + one collateral section), not just a single collateral section. Updated configuration dimensions table, user flow diagram, feature inventory description, and Phase A runtime description.
2. **FEATURE_product-type-builder.md** — redesigned Step 1 from "Select Collateral Section" (single) to "Select Application Sections" (multi-section). Updated user story, scope, Mermaid workflow, AC-1 (codename validation), AC-2 split into AC-2a (standard sections) and AC-2b (collateral section), AC-5 (preview shows all sections), AC-6 (validation checks both section types). Updated Bike example steps 2.2a/2.2b.
3. **product-type-builder.html** — added Step 0 "Create Product Type" screen with name, codename, description, version, status fields. Redesigned Step 1 with two section groups: standard sections (multi-select table with checkboxes) and collateral section (single-select cards). Added template summary card. Updated Step 4 preview to show all 4 sections. Updated Step 5 summary to include application sections row.

### Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Standard sections as multi-select | PO toggles on/off | Different product types may need different customer data (e.g., skip References for simple products) |
| Collateral section as single-select | One per product type | A product type is defined by its collateral type — mixing bike + car in one product type is architecturally unsound |
| Step 0 as explicit screen | Separate from Step 1 | User feedback: "I don't see a step to create product type." Creating the DRAFT record is a distinct action from selecting sections |
| Standard section list is system-managed | Engineering creates, PO selects | Same pattern as collateral sections — reduces PO complexity, ensures field quality |

---

## Update: 2026-03-16 — Section + Variant Model

### What Changed

User clarification: **all sections can have more than one style, called a "variant."** For example, the Collateral section has variants (Car, Bike, Tractor, Land), but so does Identity (Thai National, Foreigner, Corporate), Address (Standard, Rural), Occupation (Employed, Self-Employed, Freelance), etc.

This replaces the previous two-category model (standard sections vs. collateral section) with a **unified section + variant model**: every section works the same way — PO selects which sections to include, then picks a variant for each.

### Updated Documents

1. **CAPABILITY.md** — replaced "Application Template: Multi-Section Composition" with "Sections & Variants". Added full Section & Variant Registry table. Updated configuration dimensions, user flow diagram, and runtime Phase A description.
2. **FEATURE_product-type-builder.md** — updated user story, scope, Mermaid workflow Step 1, AC-2a/2b (section toggle + variant selection), AC-5/AC-6 (variant-aware preview and validation), Bike example step 2.2.
3. **product-type-builder.html** — replaced standard sections table + collateral cards with unified section table. Each row has: include checkbox, section name, variant dropdown, field count badge. Collateral row is always checked (required). Variant dropdown updates field preview. Updated JS: `toggleSection()`, `changeVariant()`, `updateTemplateSummary()`.

### Design Decision

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Unified section + variant model | All sections follow the same pattern | User clarified: "all sections can have more than 1 style called variant." Collateral is not special — it's just one section among many, each with its own variants. |
| Collateral section always required | Cannot be toggled off | A product type without a collateral section is architecturally invalid for a collateral-backed loan |
| One variant per section | Exactly one | A section defines a field set — mixing two variants of the same section (e.g., Thai National + Foreigner identity) doesn't make sense |

---

## Update: 2026-03-16 — Smart Form as Single Source of Truth for Sections & Variants

### What Changed

Reconciled section & variant definitions between Smart Form and Product Type Configuration. Smart Form is now the **single source of truth** for all section/variant field definitions. Product Type Configuration references Smart Form's registry — no duplicate field tables.

### Updated Documents

1. **Smart Form CAPABILITY.md** — renamed "Collateral Section Variants" → "Section & Variant Registry". Added 12 non-collateral variant definitions with full field specs (Identity: Thai National/Foreigner/Corporate, Address: Standard/Rural, Occupation: Employed/Self-Employed/Freelance, Income: Standard/Detailed, References: Standard/With Guarantor). Updated Section Selection Rule from "campaign selects" to "product type selects sections + variants". Added Stage-to-Section mapping table.
2. **Product Type Config CAPABILITY.md** — removed duplicate Section & Variant Registry table, replaced with cross-reference to Smart Form. Replaced "Collateral Section: Engineering-Owned" with "Section Variants: Engineering-Owned" referencing Smart Form. Updated Dimensions table to link to Smart Form's registry.
3. **FEATURE_product-type-builder.md** — AC-2a/2b updated to reference Smart Form's Section & Variant Registry as data source. Added Smart Form as dependency.
4. **CHANGELOG_007** — this entry.
5. **BACKLOG.md** — session log updated.

### Design Decision

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Smart Form owns all section/variant field definitions | Single source of truth | Prevents drift between capabilities. Field specs (names, types, Thai labels, validation) change in one place. Product Type Config only stores which sections/variants are selected. |
| Product Type Config references Smart Form | Cross-reference, not duplication | Smart Form already had collateral section specs. Extending it with standard sections keeps all field definitions co-located. |
| Section Selection Rule updated | "Product type selects" replaces "campaign selects" | Product type is assembled by PO before any campaign exists. Campaign references the product type — it does not select sections directly. |
