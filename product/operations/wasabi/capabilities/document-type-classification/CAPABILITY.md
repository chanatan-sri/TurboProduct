# Capability: Document Type Classification

**Product**: Wasabi — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (AI/ML Team)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Identify the type of document in the uploaded image and compare it against the expected document type. Flag type mismatches immediately with a suggested correction — enabling underwriters to re-upload the correct document before proceeding.

## Why It Exists (First Principles)

- **Early Mismatch Detection**: Document type mismatches (uploading a vehicle registration instead of a Thai ID card) are among the most common errors. Detecting this at upload prevents the wrong document from traveling all the way through the underwriting workflow to QA review.
- **Actionable Suggestion**: Detecting the mismatch is not enough — the system must suggest what document type was actually detected, so the underwriter knows what to re-upload.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Document Type Identifier | Draft | LLM identifies document type from image (e.g., THAI_ID_CARD, CAR_FRONT_PHOTO, HOUSE_DEED) |
| Type Match Comparison | Draft | Compares detected type against expected documentTypeKey |
| Type Mismatch Flag | Draft | On mismatch: flag document as requires_human_review; include suggestedType in report |
| Confidence Gate | Draft | If classification confidence < 99.99%: flag as uncertain regardless of type match |

---

## Business Rules

### Classification Rules

| Condition | Action |
|-----------|--------|
| Detected type matches expected `documentTypeKey`, confidence ≥ 99.99% | Proceed to Stage 3 (Instruction Verification) |
| Detected type does not match expected `documentTypeKey` | Flag requires_human_review; include suggestedType in report |
| Confidence < 99.99% (regardless of type) | Flag requires_human_review; uncertain classification |

### Mismatch Communication to Underwriter

When type mismatch detected, Onigiri surfaces:
> *"⚠️ Expected: บัตรประจำตัวประชาชน — Detected: ทะเบียนรถ"*

Underwriter can re-upload the correct document before proceeding.

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| suggestedType always included on mismatch | Report must include what was detected, not just that it was wrong |
| Confidence gate | Type match is insufficient alone — classification confidence must be ≥ 99.99% to proceed |
