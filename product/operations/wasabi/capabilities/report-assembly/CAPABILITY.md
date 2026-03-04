# Capability: Report Assembly

**Product**: Wasabi — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (AI/ML Team)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Aggregate all stage results (quality assessment, type classification, per-item instruction verification) into a single, structured per-document verification report returned to Onigiri — containing everything needed for underwriter warning display (Phase 1) and Matcha routing decisions (Phase 2).

## Why It Exists (First Principles)

- **Single Contract**: Onigiri needs one structured report per document that answers all three core questions (type correct? data correct? instructions completed?). Report assembly is the aggregation layer that produces this contract.
- **Downstream Routing**: Matcha Verification Router needs a structured, machine-readable result to make routing decisions. The report provides this in a consistent format regardless of which stages produced which results.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Stage Result Aggregator | Draft | Combines quality, type classification, and per-item results into one report |
| Human Review Flag | Draft | Sets requires_human_review: true if any stage produced a non-confident result |
| Summary Counts | Draft | Counts: total items, matched items, mismatched items, uncertain items, failed policy items |
| Overall Confidence | Draft | Aggregate confidence score across all items (used by Matcha Router) |
| Idempotent Caching | Draft | Same requestId returns cached report without re-processing |

---

## Business Rules

### Report Structure

```
DocumentVerificationReport {
  documentId            // Onigiri document ID
  requestId             // For idempotency caching
  quality {
    score               // 0–100
    usable              // boolean
    failure_reason?     // if not usable
  }
  type_classification {
    expected_type       // documentTypeKey from request
    detected_type       // LLM-identified type
    type_match          // boolean
    suggested_type?     // if mismatch
    confidence          // 0–1
  }
  items [
    {
      check_name        // field key (data items) or policy ID (policy items)
      item_type         // data | policy
      status            // match | mismatch | low_confidence | not_extractable |
                        // instruction_completed | instruction_failed | instruction_uncertain
      extracted_value?  // for data items
      system_value?     // for data items (from request)
      reasoning?        // LLM reasoning for uncertain or failed items
      confidence        // 0–1
    }
  ]
  summary {
    total_items
    matched_items
    mismatched_items
    uncertain_items
    failed_policy_items
    overall_confidence  // min confidence across all items
    requires_human_review  // boolean: true if any item is not fully confident
    human_review_reason?   // first reason found (type mismatch / low confidence / etc.)
  }
}
```

### Human Review Flag Rules

`requires_human_review: true` if ANY of the following:
- Image quality unusable
- Type mismatch or type classification confidence < 99.99%
- Any data item status is `mismatch`, `low_confidence`, or `not_extractable`
- Any policy item status is `instruction_failed` or `instruction_uncertain`

### Idempotency Rule

Same `requestId` returns cached report without re-processing. This prevents duplicate LLM calls if Onigiri retries the same request.

### Re-processing Rule

If the underwriter re-uploads a document or updates application data, a new `requestId` is sent. Wasabi re-processes with the new data. Latest results are authoritative.

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Complete report always returned | Even on quality failure or type mismatch, a complete report with available data is returned |
| Idempotency | Same requestId must return same (cached) result without re-processing |
| Structured output | Report must be machine-readable JSON conforming to the defined contract |
