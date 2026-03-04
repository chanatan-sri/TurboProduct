# Capability: Instruction Verification

**Product**: Wasabi — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (AI/ML Team)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Verify each document against the verification instructions fetched from Matcha configuration — extracting field values for data items and evaluating natural-language instructions for policy items — producing a per-item result with confidence and reasoning.

## Why It Exists (First Principles)

- **Config-Driven Verification**: Verification instructions are owned by Matcha (Operations), not Wasabi. Fetching them at runtime ensures Wasabi always uses the most current rules without any code changes or redeployment.
- **Data Items**: Structured field comparisons (name, ID number, date) can be done by LLM with high accuracy and clear confidence scoring.
- **Policy Items**: Subjective checks ("is the signature genuine?") are expressed as natural-language instructions. The LLM evaluates these as an inspector would, using the check_description as its instruction.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Instruction Fetcher | Draft | Fetch DocumentVerificationItem config from Matcha API per documentTypeKey before evaluation |
| Data Item Extractor | Draft | LLM extracts field value from document image for each data item; compares against system data jsonb value |
| Policy Item Evaluator | Draft | LLM reads check_description as natural-language instruction; evaluates pass/fail against document image |
| Per-Item Result Builder | Draft | Produces structured result per item with status, extracted value, confidence, and reasoning |
| Config Caching | Draft | Cache DocumentVerificationItem per documentTypeKey to avoid repeated Matcha API calls |

---

## Business Rules

### Data Item Result Statuses

| Status | Meaning |
|--------|---------|
| `match` | Extracted value matches system value. Confidence ≥ 99.99%. |
| `mismatch` | Values differ. Highlighted for underwriter + downstream verifier. |
| `low_confidence` | Confidence < 99.99%. Cannot determine match reliably. |
| `not_extractable` | Field could not be extracted (obscured, missing, unreadable). |

### Policy Item Result Statuses

| Status | Meaning |
|--------|---------|
| `instruction_completed` | Instruction satisfied. Confidence ≥ 99.99%. |
| `instruction_failed` | Instruction NOT satisfied. Highlighted for attention. |
| `instruction_uncertain` | LLM cannot determine pass/fail with sufficient confidence. |

### Routing Implications

Any item result other than `match` or `instruction_completed` causes the downstream consumer (Matcha Verification Router) to route the task to human QA.

### Config-Driven Rule

When a policy item's `check_description` is updated in the Matcha database, Wasabi automatically uses the updated instruction on the next processing call. Zero prompt engineering, zero deployment.

### LLM Call Strategy

- One LLM call per document (all items in one prompt) to minimize API cost
- Thai-first: all field names, instructions, and values must be handled in Thai script natively
- Structured output: LLM returns structured JSON results; Wasabi parses and validates

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Zero-deployment rule updates | New/changed verification instructions in Matcha must be automatically picked up |
| Single LLM call per document | All items for a document processed in one call to minimize cost |
| Thai script support | LLM must natively handle Thai script for field names, values, and instructions |
| Confidence required per item | Every item result must include a confidence score |

---

## Open Questions

- What is the config caching TTL for DocumentVerificationItem? How quickly do Matcha config changes need to be reflected?
- Multi-page documents: single LLM call with all pages or separate calls per page?
- Should a consensus voting strategy (multiple LLM calls + majority) be used for higher confidence on borderline items?
