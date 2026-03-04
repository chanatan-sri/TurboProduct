# Capability: Flexible Logic Configuration

**Product**: Matcha — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: ✅ Active — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Allow Operations to define and change document verification rules without requiring engineering code deployments. Provides two types of verification checks (data items and policy items), per-document correct/incorrect decisions, save-as-you-go persistence, and task-level outcome aggregation.

## Why It Exists (First Principles)

- **Operational Agility**: Document types and verification requirements change frequently. New document types are added, new fields become required, policy standards evolve. If every rule change requires a code deployment, Operations cannot respond to business changes in time.
- **Separation of Concerns**: Rules are business logic, not code. Storing them in the database allows product owners to manage them without engineering involvement.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Data Item Configuration | Live | Configure field-level matching checks: check_name maps to document data key; verifier compares document image against system value |
| Policy Item Configuration | Live | Configure subjective boolean checks: check_description is the natural-language instruction; verifier assesses pass/fail |
| Per-Document Correct/Incorrect Decision | Live | Verifier marks each document as Correct or Incorrect with pre-configured rejection_reason and free-text remark |
| Save-as-you-go Persistence | Live | Each document decision saved independently; progress survives browser close |
| Task Outcome Aggregation | Live | APPROVED (all correct) / RETURNED (any incorrect) / REFERRED (task-level escalation) |
| Submission Guardrails | Live | System validates all required decisions made before enabling submission; confirmation dialog prevents accidental submission |

---

## Business Rules

### Two Types of Check Items

| Type | item_type | How It Works | Used For |
|------|-----------|-------------|----------|
| **Data Item** | `data` | `check_name` maps to a key in document `data` jsonb. Verifier compares document image against the system value for that key. | Field-level data accuracy (name, ID number, plate number, etc.) |
| **Policy Item** | `policy` | `check_description` is a natural-language instruction for the verifier (or LLM). Verifier assesses pass/fail subjectively. | Non-data subjective checks (signature legible, document not expired, no visible tampering) |

### Per-Document Decision Rules

| Decision | Condition | Consequence |
|----------|-----------|-------------|
| Correct | All check items pass | Document marked correct. Contributes to APPROVED outcome. |
| Incorrect | Any check item fails | Pre-configured `error_message` shown; `rejection_reason` stored; free-text `remark` available. Contributes to RETURNED outcome. |

### Task-Level Outcome Rules

| Outcome | Condition |
|---------|-----------|
| APPROVED | All documents marked Correct |
| RETURNED | Any document marked Incorrect |
| REFERRED | Task-level escalation by verifier (not per-document; supersedes document-level outcome) |

### Save-as-you-go Rules

- Each document save persists: document-level decision (Correct/Incorrect), all check item outcomes (correct/incorrect or pass/fail), remarks
- Saves are independent per document — verifier can save one document and move to the next without submitting the task
- Progress survives browser close — task re-opens to last saved state
- The pre-configured `rejection_reason` and `error_message` are stored with the result when a check item is marked incorrect

### Submission Guardrail Rules

- Submission button is disabled until all documents have a decision recorded
- A confirmation dialog appears on submission to prevent accidental misclicks
- Dynamic rules are stored as database configuration — no code change needed for new document types

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| No-code rule updates | New document types and verification rules require no code deployment |
| Save atomicity | Each document save is atomic — all check items + document decision persisted in a single write |
| Session resilience | In-progress verification state survives browser close and page reload |
