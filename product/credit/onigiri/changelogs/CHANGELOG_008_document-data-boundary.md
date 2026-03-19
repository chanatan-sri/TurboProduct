# CHANGELOG 008: Onigiri/Matcha Document Data Boundary

**Date**: 2026-03-17
**Layer**: Cross-Product (Onigiri + Matcha)
**Type**: Boundary Clarification + Feature Design

---

## What Changed

### 1. Documented the Onigiri/Matcha Document Verification Boundary

Clarified who owns what in document verification as a **three-way contract**:

| Concern | Owner | What They Configure |
|---------|-------|-------------------|
| **What to verify** (instructions) | Matcha / Operations | `DocumentVerificationItem` records: data items (`check_name`) and policy items (`check_description`, e.g., "มีลายเซ็น", "ข้อมูลตรงกับในระบบ") |
| **Which fields to send** (extraction) | Onigiri / Engineering | Data Extraction Templates: `check_name` → application field path |
| **Which documents are required** (declarations) | Onigiri / PO | Document requirement declarations per product type |

### 2. Introduced Data Extraction Templates

Replaced the vague "JSONPath extraction" concept with a precise **Data Extraction Template** model:

- Engineering-owned pre-configured mappings from application field paths to Matcha `check_name` keys
- One template per document type (e.g., `thai_id_card_standard`)
- PO selects a template when declaring document requirements — does not configure field mappings
- Consistent with existing pattern: Engineering creates building blocks (section variants, extraction templates), PO assembles

### 3. Updated Matcha Boundary Documentation

- Matcha PRODUCT.md: Added "IS NOT responsible for application field extraction" to boundary
- Matcha Flexible Logic Config CAPABILITY.md: Added "Cross-System Data Flow for Data Items" section explaining that Matcha is agnostic to how clients populate the `data` jsonb

---

## Rationale

The user clarified that:
1. **Matcha** owns verification instructions — what to check on each document (e.g., "ข้อมูลตรงกับในระบบ", "ไม่มีเขียนกำกับ", "มีลายเซ็น", "ลายเซ็นถูกต้อง")
2. **Onigiri** owns knowledge of which application fields need to be verified — the actual data values (ID card number, name, etc.)

This boundary was implicit in the architecture but never explicitly documented. The "JSONPath extraction" concept in Document Requirement Declaration was a placeholder that didn't define who configures the mapping, what the mapping looks like, or how it relates to Matcha's data items.

---

## Decision Log

| Decision | Choice | Alternatives Considered | Why |
|----------|--------|------------------------|-----|
| Extraction template ownership | Engineering creates templates | PO configures field mapping; Auto-derive from naming convention | Requires cross-system knowledge (Matcha check_names + Smart Form fields). Consistent with section variant pattern. |
| Template vs. inline config | Separate reusable template entity | Inline JSON per document requirement row | Templates are reusable across product types; easier to maintain when Matcha adds new data items |
| JSONPath replacement | Named field paths referencing Smart Form field keys | Keep JSONPath syntax | Field keys are stable identifiers; JSONPath is fragile if JSON structure changes |

---

## Documents Changed

| Document | Change |
|----------|--------|
| `product/operations/matcha/PRODUCT.md` | Added boundary clarification (IS NOT + RECEIVES) |
| `product/operations/matcha/capabilities/flexible-logic-configuration/CAPABILITY.md` | Added "Cross-System Data Flow for Data Items" section |
| `product/credit/onigiri/capabilities/product-type-configuration/CAPABILITY.md` | Added three-way contract, extraction template dimension, updated dependencies |
| `product/credit/onigiri/capabilities/product-type-configuration/features/FEATURE_document-requirement-declaration.md` | Replaced all "JSONPath extraction" with Data Extraction Template concept |
| `product/credit/onigiri/backlog/ITEM_document-requirement-declaration.md` | Updated ACs and dependencies |
| `product/credit/onigiri/BACKLOG.md` | Session log entry |
