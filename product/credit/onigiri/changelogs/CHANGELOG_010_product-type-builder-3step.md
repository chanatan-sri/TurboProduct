# CHANGELOG 010: Product Type Builder — 5-Step → 3-Step Wizard Redesign

**Date**: 2026-03-18
**Layer**: @FEATURE (Product Type Builder)
**Type**: UX Redesign

---

## What Changed

### 1. Merged Wizard Steps: 5 → 3

**Previous flow (5 steps):**
Step 0: Create → Step 1: Select Sections & Variants → Step 2: Register Doc Types → Step 3: Declare Requirements → Step 4: Preview → Step 5: Submit

**New flow (3 steps):**
Step 0: Create → Step 1: Configure Sections → Step 2: Preview & Submit

Steps 2-3 (Register Doc Types + Declare Requirements) are now **inline within Step 1**, embedded in each section's accordion card. Step 4-5 (Preview + Submit) merged into a single step.

### 2. Section Accordion Cards (Step 1)

Each section in Step 1 is an expandable accordion card showing:
- **Header:** toggle on/off + section name + variant + field/doc count summary
- **Expanded:** variant dropdown + field preview (read-only) + document requirements table + "Add Doc Requirement" + "Register New Doc Type" inline
- **Conditional rules** configured per document, with trigger field validated against the section's selected variant

The Collateral section is always required (toggle locked on).

### 3. Document Requirements In-Context

Document requirements are now visible **alongside the section they serve**. When a PO selects "Collateral: Bike", they immediately see and configure bike-specific documents (registration book, insurance, DLT web page) in the same card. This eliminates the disconnected feeling of configuring documents in a separate step.

### 4. Updated Mock UI

product-type-builder.html rebuilt with:
- 4 screens: Create → Configure Sections → Preview & Submit → Approval Status
- Accordion cards with expand/collapse, variant switching, inline doc config
- Pre-populated Bike Title example (4 sections, 54 fields, 4 documents)
- Conditional rule simulation in Preview tab

---

## Rationale

The original 5-step flow separated section selection from document configuration, creating a disconnect. POs had to mentally map "which documents go with which section" across separate steps. The redesign puts documents in context — when you expand the Collateral: Bike section, you see 17 fields AND the 3 required documents together. This reduces cognitive load and prevents configuration errors (e.g., declaring a document requirement for a field that doesn't exist in the selected variant).

---

## Decision Log

| Decision | Choice | Alternatives Considered | Why |
|----------|--------|------------------------|-----|
| Step merge strategy | Merge docs into section cards | Keep separate steps but reorder; two-panel layout | Accordion cards keep the full picture per section without overloading — PO only sees expanded content for the section they're working on |
| Document requirements ownership | Per-section (shown in section card) | Per-product-type (global list) | Documents are semantically tied to sections (identity docs go with Identity section, collateral docs with Collateral). Per-section placement is more intuitive. |
| Preview & Submit merge | Single step with tabs | Keep as separate steps | Preview is a validation gate, not a standalone configuration step. Merging reduces clicks. |

---

## Documents Changed

| Document | Change |
|----------|--------|
| `FEATURE_product-type-builder.md` | Rewrote workflow (3-step Mermaid), added Section Configuration Card UI pattern, updated ACs, updated Bike Title example |
| `product-type-configuration/CAPABILITY.md` | Updated user flow Mermaid diagram + Feature Inventory description |
| `product-type-builder.html` | Rebuilt mock UI with accordion cards, inline doc config |
| `BACKLOG.md` | Updated Last Session + session log |
