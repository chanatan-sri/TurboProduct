# Capability: Universal Document Verification Engine

**Product**: Matcha — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: ✅ Active — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

A single, unified service for all document verification needs across the enterprise (Loans, Insurance, KYC). Processes "Tasks" and "Documents" based on configuration — with no knowledge of the originating domain. Maintains a universal 4-state task lifecycle and integrates with the Solomon QA worklist via work-entry/work-done events.

## Why It Exists (First Principles)

- **Domain Fragmentation**: Previously, each business domain required its own verification implementation with its own audit trail, QA distribution, and rule configuration. This created duplicated operational overhead and inconsistent verification quality across products.
- **Single Audit Trail**: A complete, immutable audit trail for all verification activities across the company requires a single system of record — not one per domain.
- **Reusability**: Verification logic (data field matching, policy subjective checks, change detection) is the same regardless of whether the document is a Thai ID card for a loan or an insurance policy — only the rules differ.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Task Lifecycle Manager | Live | PENDING → IN_PROGRESS → COMPLETED lifecycle with PENDING_REVIEW re-entry |
| Solomon Integration (work-entry/work-done) | Live | Publish SQS work-entry when task needs QA; work-done when task completes |
| Task Creation API | Live | POST /task endpoint accepting documents, verification config, and callback URL from client systems |
| Outcome Webhook Callback | Live | POST to client callbackUrl on task COMPLETED with APPROVED/RETURNED/REFERRED outcome |
| PENDING_REVIEW Re-entry | Live | Transition COMPLETED task back to PENDING_REVIEW when late data arrives or context changes |

---

## Business Rules

### 4-State Task Lifecycle

| State | Meaning | Entry | Exit |
|-------|---------|-------|------|
| PENDING | Task created, awaiting QA distribution | Task created via POST /task | work-entry published to Solomon |
| IN_PROGRESS | Verifier has opened the task in Solomon | Verifier opens task URL | Verifier submits decision OR AI auto-completes |
| COMPLETED | Verification decision made and callback sent | Verifier submits / AI auto-complete | If late data arrives → PENDING_REVIEW |
| PENDING_REVIEW | Late data arrived or change detected post-decision | context_hash changed after COMPLETED OR car check late data | Verifier re-submits → COMPLETED again (is_re_decision: true) |

### Outcome as Data Model

The outcome (APPROVED / RETURNED / REFERRED) is stored **as data** on the TaskCompletionEvent — NOT as a separate workflow state. This ensures the state machine is reusable across all task types without adding domain-specific states.

| Outcome | Meaning |
|---------|---------|
| APPROVED | All documents correct |
| RETURNED | Any document incorrect — sent back to task originator for correction |
| REFERRED | Task-level escalation to senior approver for policy exceptions |

### PENDING_REVIEW Re-entry Rules

PENDING_REVIEW is entered when:
1. Late car check data arrives after a COMPLETED decision
2. context_hash of a document changes after a COMPLETED decision (data or file URLs changed)

On re-entry: a new SQS work-entry is published so the task reappears in Solomon. The verifier retains full authority to change, confirm, or escalate the decision. A new TaskCompletionEvent is created with `is_re_decision: true`. The old event is never mutated.

---

## Task Lifecycle Diagram

```mermaid
stateDiagram-v2
    [*] --> PENDING: POST /task (client creates task)
    PENDING --> IN_PROGRESS: Verifier opens task\n(work-entry consumed from Solomon)
    IN_PROGRESS --> COMPLETED: Verifier submits decision\nOR AI auto-complete
    COMPLETED --> PENDING_REVIEW: Late car check data\nOR context_hash changed
    PENDING_REVIEW --> IN_PROGRESS: New work-entry to Solomon;\nverifier re-opens
    COMPLETED --> [*]: Outcome webhook → client
```

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Domain agnosticism | System must not contain domain-specific logic (no "loan" or "insurance" concepts) |
| Single audit trail | Every verification decision (human or AI) must produce an immutable TaskCompletionEvent |
| Solomon integration | All QA-bound tasks must publish work-entry to Raijin/Hephaestus; no direct assignment |
| Callback reliability | Outcome webhook must be delivered to client callbackUrl; retry logic required on failure |
