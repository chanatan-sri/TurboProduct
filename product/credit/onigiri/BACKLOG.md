# BACKLOG: Loan Origination System

**Product:** Onigiri — Loan Origination System
**Portfolio:** Credit
**Last Updated:** 2026-03-30
**Last Session:** Added Insurance Integration capability (5 features + 1 cross-capability item). Two insurance types: Credit Insurance (plan retrieval from external API) and Voluntary/Compulsory Insurance (reference number lookup). Ontop/Deduct premium logic based on campaign budget threshold (max_credit_line × insurance_budget_pct). Updated Campaign Config, Smart Form, Product Type Config, and PRODUCT.md.

---

## ✅ LIVE
| Feature | Capability | Shipped | Item File |
|---|---|---|---|

## 🔨 IN PROGRESS
| Feature | Capability | Owner | Started | Item File |
|---|---|---|---|---|

## 📋 SPEC'D (ready to build)
| Feature | Capability | Priority | Item File |
|---|---|---|---|

## 💡 CONCEPT (not yet specced)
| Feature | Capability | Item File |
|---|---|---|
| Collateral Section Registry | Product Type Configuration | [ITEM](backlog/ITEM_collateral-section-registry.md) |
| Document Requirement Declaration | Product Type Configuration | [ITEM](backlog/ITEM_document-requirement-declaration.md) |
| Document Type Registration | Product Type Configuration | [ITEM](backlog/ITEM_document-type-registration.md) |
| Product Type Publication Authorization | Product Type Configuration | [ITEM](backlog/ITEM_product-type-publication-authorization.md) |
| Product Type Builder | Product Type Configuration | [ITEM](backlog/ITEM_product-type-builder.md) |
| Application Template Assignment | Loan Campaign Configuration | [ITEM](backlog/ITEM_application-template-assignment.md) |
| DipChip Integration Gate | AI-Assisted Data Entry | [ITEM](backlog/ITEM_dipchip-integration-gate.md) |
| Document Upload with AI Extraction | AI-Assisted Data Entry | [ITEM](backlog/ITEM_document-upload-ai-extraction.md) |
| Extraction Field Mapping Registry | AI-Assisted Data Entry | [ITEM](backlog/ITEM_extraction-field-mapping-registry.md) |
| Smart Form Pre-fill Engine | AI-Assisted Data Entry | [ITEM](backlog/ITEM_smart-form-prefill-engine.md) |
| Pre-fill Conflict Resolution | AI-Assisted Data Entry | [ITEM](backlog/ITEM_prefill-conflict-resolution.md) |
| Extraction Confidence Display | AI-Assisted Data Entry | [ITEM](backlog/ITEM_extraction-confidence-display.md) |
| Multi-Document Pre-fill Orchestration | AI-Assisted Data Entry | [ITEM](backlog/ITEM_multi-document-prefill.md) |
| Credit Insurance Plan Retrieval | Insurance Integration | [ITEM](backlog/ITEM_credit-insurance-plan-retrieval.md) |
| External Insurance Reference Lookup | Insurance Integration | [ITEM](backlog/ITEM_external-insurance-reference-lookup.md) |
| Insurance Premium Ontop/Deduct Calculator | Insurance Integration | [ITEM](backlog/ITEM_insurance-premium-ontop-deduct-calculator.md) |
| Insurance Section in Smart Form | Insurance Integration | [ITEM](backlog/ITEM_insurance-section-smart-form.md) |
| Insurance Data Pass-through to Plan Calculation | Insurance Integration | [ITEM](backlog/ITEM_insurance-plan-calculation-passthrough.md) |
| Campaign Insurance Budget Configuration | Loan Campaign Configuration | [ITEM](backlog/ITEM_campaign-insurance-budget-config.md) |

## 🗄️ DEPRECATED
| Feature | Capability | Date | Item File |
|---|---|---|---|

---

## SESSION LOG
| Date | Type | Summary | Docs Changed |
|---|---|---|---|
| 2026-03-10 | @CAPABILITY | Added Product Type Configuration capability (4 features) to enable zero-code collateral type definition. Design decisions: Onigiri owns doc type registry, same 2-tier approval, template-based builder. | CAPABILITY.md (new), 4x FEATURE_*.md (new), 4x ITEM_*.md (new), BACKLOG.md (new), PRODUCT.md (updated), CHANGELOG_006 |
| 2026-03-11 | @FEATURE | Refined ownership: Collateral Section Builder → Collateral Section Registry. Engineering creates sections; PO selects. Reduces user complexity — POs don't build 17-60+ field forms. | CAPABILITY.md, FEATURE_collateral-section-registry.md (renamed), ITEM_collateral-section-registry.md (renamed), BACKLOG.md, CHANGELOG_006 |
| 2026-03-12 | @FEATURE | Added Product Type Builder orchestration feature. 5-step wizard (Select Section → Register Doc Types → Declare Requirements → Preview → Submit). Includes Campaign Configuration integration (AC-11–13) and concrete Bike Title Loan example. | FEATURE_product-type-builder.md (new), ITEM_product-type-builder.md (new), CAPABILITY.md (updated), BACKLOG.md (updated), CHANGELOG_007 |
| 2026-03-16 | @CAPABILITY | Documented 5-phase CO runtime flow: Product Type → Data Entry → Campaign Matching (with person/product limits) → Financial Details → Document Upload → Send to Approver. Added "Product Type as Application Entry Point" business rule to CAPABILITY.md. Created CO application mock UI (Bike example). Updated Bike example Phase 5 with 13-step runtime. | CAPABILITY.md (updated), FEATURE_product-type-builder.md (updated), co-application-flow.html (new), BACKLOG.md (updated), CHANGELOG_007 (updated) |
| 2026-03-16 | @FEATURE | Redesigned Product Type Builder: (1) Added Step 0 "Create Product Type" screen. (2) Introduced section + variant model — every section has variants (e.g., Identity: Thai National/Foreigner/Corporate, Collateral: Bike/Car/Tractor/Land). PO selects sections and picks one variant per section. Unified table UI with variant dropdowns. | CAPABILITY.md (updated), FEATURE_product-type-builder.md (updated), product-type-builder.html (updated), BACKLOG.md (updated), CHANGELOG_007 (updated) |
| 2026-03-16 | @CAPABILITY | Reconciled Smart Form as single source of truth for all section/variant definitions. Added 12 non-collateral variants with full field specs to Smart Form. Updated Section Selection Rule (product type selects, not campaign). Added Stage-to-Section mapping. Removed duplicate registry from Product Type Config — replaced with cross-references to Smart Form. | Smart Form CAPABILITY.md (updated), Product Type Config CAPABILITY.md (updated), FEATURE_product-type-builder.md (updated), BACKLOG.md (updated), CHANGELOG_007 (updated) |
| 2026-03-17 | @CAPABILITY | Documented Onigiri/Matcha document verification boundary as three-way contract (Matcha verification instructions / Onigiri extraction templates / PO document declarations). Introduced Data Extraction Templates — engineering-owned mappings from application fields to Matcha `check_name` keys. Replaced "JSONPath extraction" with precise template concept. Updated Matcha boundary docs. | Matcha PRODUCT.md, Matcha Flexible Logic Config CAPABILITY.md, Product Type Config CAPABILITY.md, FEATURE_document-requirement-declaration.md, ITEM_document-requirement-declaration.md, CHANGELOG_008 (new), BACKLOG.md |
| 2026-03-18 | @FEATURE | Created Campaign ↔ Product Type integration spec: Application Template Assignment feature (PM selects ACTIVE product type → sections + docs inherited read-only). Added Application Type eligibility field (new_booking/topup/restructure). Added product type version pinning rule. Corrected: collateral type remains independent eligibility criterion. | FEATURE_application-template-assignment.md (new), ITEM_application-template-assignment.md (new), Campaign CAPABILITY.md (updated), Product Type Config CAPABILITY.md (updated), BACKLOG.md (updated), CHANGELOG_009 (new) |
| 2026-03-18 | @FEATURE | Redesigned Product Type Builder: 5-step → 3-step wizard. Merged document type registration + requirement declaration into Configure Sections step (inline per section accordion card). PO sees fields + documents together in context. Updated mock UI with accordion cards. | FEATURE_product-type-builder.md (updated), Product Type Config CAPABILITY.md (updated), product-type-builder.html (rebuilt), BACKLOG.md (updated), CHANGELOG_010 (new) |
| 2026-03-18 | @CAPABILITY | Introduced 3-level document hierarchy: Document Type (AI classification, Engineering-owned) → Evidence (what customer brings, PO per section) → Upload Box (upload slots with order, allow multiple, min/max, tooltip). Document Type Registration shifted to Engineering ownership. Same doc type can map to multiple evidence entries. Two evidence categories: customer/guarantor docs + system-generated (Contract, PDPA, Insurance — always same). | FEATURE_document-type-registration.md (rewritten), FEATURE_document-requirement-declaration.md (rewritten), CAPABILITY.md (updated), FEATURE_product-type-builder.md (updated), BACKLOG.md (updated), CHANGELOG_011 (new) |
| 2026-03-30 | @CAPABILITY | Added AI-Assisted Data Entry capability with 7 features. Extended Wasabi with extraction mode (dual-mode: verification + extraction). DipChip gate → upload → Wasabi extraction → mapping registry → pre-fill engine → conflict resolution → confidence display → multi-document orchestration. Reuses existing extraction_template table bidirectionally. Updated Wasabi boundary, Instruction Verification, Report Assembly. | AI-Assisted Data Entry CAPABILITY.md (new), 7x FEATURE_*.md (new), 7x ITEM_*.md (new), Wasabi PRODUCT.md (updated), Wasabi Instruction Verification CAPABILITY.md (updated), Wasabi Report Assembly CAPABILITY.md (updated), Onigiri PRODUCT.md (updated), BACKLOG.md (updated), CHANGELOG_012 (new) |
| 2026-03-30 | @CAPABILITY | Added Insurance Integration capability with 5 features + 1 cross-capability item. Two insurance types: Credit Insurance (external API plan retrieval) and Voluntary/Compulsory Insurance (reference number lookup). Ontop/Deduct logic: total_premium vs campaign budget (max_credit_line × insurance_budget_pct). Both types can coexist; credit insurance optional; multiple voluntary references allowed. Insurance fields locked at Approval HWM. | Insurance Integration CAPABILITY.md (new), 5x FEATURE_*.md (new), 6x ITEM_*.md (new), Campaign Config CAPABILITY.md (updated), Smart Form CAPABILITY.md (updated), Product Type Config CAPABILITY.md (updated), PRODUCT.md (updated), BACKLOG.md (updated), CHANGELOG_013 (new) |
