# Capability: Document Quality Assessment

**Product**: Wasabi — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (AI/ML Team)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Assess document image quality before any extraction or verification attempt. Filter out unusable images (blurred, poorly lit, truncated, obstructed) to avoid expensive LLM calls on images that cannot be reliably read — and to surface actionable quality feedback to the underwriter immediately.

## Why It Exists (First Principles)

- **Cost Gate**: Running LLM extraction on an unusable image wastes API cost and produces unreliable results. Early quality filtering prevents downstream processing on inherently unverifiable documents.
- **Immediate Feedback**: Underwriters upload documents directly. Immediate quality feedback ("re-upload a clearer image") allows same-session correction rather than discovering the issue in QA days later.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Image Quality Evaluator | Draft | LLM evaluates image on blur, lighting, truncation, obstruction, watermarks |
| Quality Score | Draft | Numeric quality score 0–100 included in per-document report |
| Unusable Image Gate | Draft | If quality below threshold → skip remaining stages; flag as requires_human_review |
| Quality Failure Reason | Draft | Human-readable reason returned to Onigiri for display to underwriter (e.g., "Image blurry — re-upload") |

---

## Business Rules

### Quality Dimensions Assessed

| Dimension | Description |
|-----------|-------------|
| Blur | Image is too blurry to read text or data |
| Lighting | Image too dark, overexposed, or uneven lighting obscuring content |
| Truncation | Document edges cut off — required fields not fully visible |
| Obstruction | Fingerprints, objects, or stickers covering part of the document |
| Watermarks | Watermarks obscuring data fields |

### Quality Gate Rule

If image quality score is below the usability threshold → skip Stages 2, 3, 4. Return:
- `requires_human_review: true`
- `human_review_reason: "Image quality insufficient for AI verification"`
- `quality_score` in report

### Cost Optimization Rule

Quality assessment uses the lightest/cheapest LLM evaluation to serve as a gate before the more expensive instruction verification stage.

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Early exit | Quality failure must prevent downstream LLM calls |
| Actionable reason | Quality failure reason must be human-readable for underwriter display |
| Score included in report | Quality score 0–100 always included in report regardless of outcome |
