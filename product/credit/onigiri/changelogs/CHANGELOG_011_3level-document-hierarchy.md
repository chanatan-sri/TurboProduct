# CHANGELOG 011: 3-Level Document Hierarchy — Document Type → Evidence → Upload Box

**Date**: 2026-03-18
**Layer**: @CAPABILITY (Product Type Configuration) + @FEATURE (Document Type Registration, Document Requirement Declaration, Product Type Builder)
**Type**: Model Redesign

---

## What Changed

### 1. Introduced 3-Level Document Hierarchy

Replaced the flat "Document Type = Document Requirement" model with a 3-level hierarchy that reflects how the real system works:

| Level | What It Is | Owner | Example |
|-------|-----------|-------|---------|
| **Document Type** | AI classification category | **Engineering** | `copy_of_id_card`, `vehicle_registration_book` |
| **Evidence** | What customer must bring — per section | **PO** | "สำเนาบัตรประชาชนผู้กู้", "สมุดทะเบียนรถ" |
| **Upload Box** | Specific upload slot within an evidence | **PO** | "หน้าปก", "หน้ากรรมสิทธิ์ล่าสุด" |

### 2. Document Type Registration → Engineering-Owned

**Previous:** PO registers document types in Admin UI.
**New:** Engineering registers document types. Document types are AI classification categories — creating them requires understanding Matcha's verification capabilities and AI model boundaries. PO references existing types when declaring evidence.

### 3. Document Requirement Declaration → Evidence + Upload Box Configuration

**Previous:** PO declares "document requirements" (flat: one document type = one requirement).
**New:** PO declares **evidence** (what customer brings) at the section level, then configures **upload boxes** per evidence with:
- Order in Upload Group
- Allow Multiple Upload (Boolean)
- Minimum files
- Maximum files
- Tooltip Message
- Tooltip Image

### 4. Same Document Type → Multiple Evidence Entries

Key insight: `copy_of_id_card` (one AI classification) can map to multiple evidence entries — "Customer ID Card" and "Guarantor ID Card" are separate evidence with separate upload boxes, but the same document type for AI classification.

### 5. Two Evidence Categories

- **Customer/Guarantor Documents** — PO-configured per section (ID cards, registration books, income docs)
- **System-Generated Documents** — Always the same across all product types (Contract, PDPA, Insurance Form). Auto-included, not configurable by PO.

### 6. Updated Accordion Card Wireframes

Product Type Builder's section accordion cards now show:
- Evidence table with document type, required status, and upload box count
- Expandable upload box configuration per evidence (order, allow multiple, min/max, tooltips)

---

## Rationale

The flat "document type = document requirement" model didn't match reality. In the real system:
- A single document type (`copy_of_id_card`) is used for both customer and guarantor — same AI classification, different evidence contexts
- Each evidence has multiple upload boxes (e.g., car registration book has 6 specific pages to upload)
- Upload boxes have configuration (order, allow multiple, min/max files, tooltip message, tooltip image)
- Document type registration is a technical task (AI classification boundaries), not a business configuration task

---

## Decision Log

| Decision | Choice | Alternatives Considered | Why |
|----------|--------|------------------------|-----|
| Document type ownership | Engineering | PO self-service (original design) | Document types define AI classification boundaries — requires understanding Matcha's verification model. PO references types, doesn't create them. |
| Evidence → Upload Box relationship | 1:N (one evidence, many upload boxes) | Flat (one evidence = one upload) | Real system has multiple upload slots per evidence (e.g., registration book has 6 pages) |
| Upload box configuration fields | Order, Allow Multiple, Min, Max, Tooltip Message, Tooltip Image | Simpler (just name) | These are the real configuration options used in the existing system |
| Conditional logic level | Evidence level (all upload boxes excluded together) | Per-upload-box | Conditional rules are about whether a piece of evidence is needed, not about individual upload slots |
| System-generated documents | Auto-included, not PO-configurable | PO toggles on/off | Contract, PDPA, Insurance Form are always the same — no reason for PO to configure |

---

## Documents Changed

| Document | Change |
|----------|--------|
| `FEATURE_document-type-registration.md` | Rewrote: Engineering-owned (not PO), added 3-level hierarchy context, updated user story and ACs |
| `FEATURE_document-requirement-declaration.md` | Rewrote: evidence + upload box model, two evidence categories, upload box config schema, car registration book example, customer/guarantor ID example |
| `product-type-configuration/CAPABILITY.md` | Added "3-Level Document Hierarchy" section, updated feature inventory (Engineering ownership for doc types), updated conditional logic to evidence level, updated three-way contract |
| `FEATURE_product-type-builder.md` | Updated accordion wireframes (evidence table + upload box config), updated Mermaid workflow, updated ACs, updated Bike Title example |
| `BACKLOG.md` | Updated Last Session + session log |
