# Business Requirements Document (BRD)
## Matcha — Document Verification Service | Phase 1

**Product**: Matcha (抹茶) — Document Verification Service
**Portfolio**: Operations
**Status**: Draft
**Version**: 1.0
**Date**: 2026-05-25
**Prepared by**: Product Management

> **Scope Note**: This BRD covers Phase 1 of Matcha. AI-First Verification (Wasabi routing, auto-complete, spot-check sampling) is deferred to Phase 2.

---

## 1. Objective

### 1.1 Business Problem

Document verification is required across multiple business domains — Loans (Onigiri), Insurance, and KYC. Prior to Matcha, each domain operated its own verification implementation, resulting in:

- Fragmented audit trails with no single source of truth for compliance
- Duplicated engineering overhead across domains
- Inconsistent verification quality and QA distribution logic
- No mechanism to detect post-decision document changes (approval leakage risk)
- Re-verification of unchanged documents on every resubmission, wasting QA capacity

### 1.2 Phase 1 Objectives

| # | Objective | Business Outcome |
|---|-----------|-----------------|
| O-1 | Establish a single, domain-agnostic document verification engine that any client system can invoke via API | Eliminate verification fragmentation; single audit trail for all domains |
| O-2 | Enable Operations to configure verification rules (data checks and policy checks) without code deployments | Reduce engineering dependency for rule changes; accelerate response to business requirement changes |
| O-3 | Integrate asynchronous vehicle data (car check) without blocking QA verifiers | Prevent QA throughput degradation due to external provider latency |
| O-4 | Detect document changes before and after a verifier's decision to prevent approval leakage | Meet regulatory and compliance requirements for verification integrity |
| O-5 | Minimize re-verification effort when a returned application is resubmitted with corrected documents | Reduce QA re-work cost; improve verifier throughput on resubmissions |

### 1.3 Out of Scope (Phase 1)

- AI-assisted verification routing (Wasabi integration, auto-complete, spot-check sampling) → **Phase 2**
- LLM document processing → owned by Wasabi
- QA worklist UX → owned by Solomon / Raijin / Hephaestus
- Loan workflow orchestration → owned by Onigiri
- Customer master data → owned by DaVinci

---

## 2. Acceptance Criteria

### 2.1 Capability: Universal Document Verification Engine

**AC-UVE-01 — Task Lifecycle**
Covers the end-to-end state progression from task creation through QA pick-up to final decision, including the outcome webhook delivery to the originating client system.
- [ ] A client system can create a verification task via `POST /task` with a document payload, verification configuration, and callback URL
- [ ] A newly created task enters `PENDING` state immediately upon creation
- [ ] When a verifier opens a task from the Solomon worklist, the task transitions to `IN_PROGRESS`
- [ ] When a verifier submits a decision (APPROVED / RETURNED / REFERRED), the task transitions to `COMPLETED`
- [ ] Upon `COMPLETED`, the system delivers an outcome webhook to the client's `callbackUrl`

**AC-UVE-02 — Solomon Integration**
Ensures all QA distribution flows through the Solomon worklist via SQS events, with no direct verifier assignment outside this channel.
- [ ] A `work-entry` SQS event is published to Raijin/Hephaestus when a task enters QA distribution
- [ ] A `work-done` SQS event is published when a task reaches `COMPLETED`
- [ ] No task is directly assigned to a verifier — all distribution is via Solomon

**AC-UVE-03 — PENDING_REVIEW Re-entry**
Governs how a closed task is reopened when new information arrives after a decision has been made, preserving full re-decision authority and audit integrity.
- [ ] When a `COMPLETED` task receives late car check data, the task transitions to `PENDING_REVIEW`
- [ ] When a `COMPLETED` task's `context_hash` changes (document data or file URLs modified after decision), the task transitions to `PENDING_REVIEW`
- [ ] On `PENDING_REVIEW`, a new `work-entry` is published to Solomon so the task reappears in the QA worklist
- [ ] The verifier retains full authority (APPROVED / RETURNED / REFERRED) during re-decision
- [ ] A re-decision creates a new `TaskCompletionEvent` with `is_re_decision: true`; the original event is never mutated
- [ ] A new outcome webhook is sent to the client with `isReDecision: true`

**AC-UVE-04 — Outcome Encoding**
Validates that outcomes are stored as data attributes rather than separate states, keeping the task lifecycle domain-agnostic and reusable across all client systems.
- [ ] Task outcomes (APPROVED / RETURNED / REFERRED) are stored as data on `TaskCompletionEvent`, not as separate lifecycle states
- [ ] The system supports all three outcomes for any task type without domain-specific code

---

### 2.2 Capability: Flexible Logic Configuration

**AC-FLC-01 — Verification Item Types**
Confirms Operations can define both structured field-match checks (Data Items) and free-form subjective checks (Policy Items) entirely through configuration with no code deployment required.
- [ ] Operations can configure **Data Items** (`item_type: data`): a `check_name` maps to a key in the document's `data` jsonb; the verifier compares the document image against the system value for that key
- [ ] Operations can configure **Policy Items** (`item_type: policy`): a `check_description` is a natural-language instruction for the verifier to assess as pass/fail
- [ ] Both item types are configurable via database and require no code deployment to add or modify

**AC-FLC-02 — Per-Document Decision**
Verifies that each document is assessed individually, with rejection details (reason, error message, remark) captured on incorrect documents and the result correctly contributing to the task-level outcome.
- [ ] A verifier can mark each document as **Correct** or **Incorrect**
- [ ] Marking a document Incorrect stores the pre-configured `rejection_reason`, `error_message`, and an optional free-text `remark`
- [ ] A document marked Correct contributes to an APPROVED task outcome
- [ ] A document marked Incorrect contributes to a RETURNED task outcome

**AC-FLC-03 — Task Outcome Aggregation**
Confirms the task-level outcome is derived correctly from document-level decisions, with REFERRED as an override path that supersedes all document outcomes regardless of their individual status.
- [ ] APPROVED: all documents are marked Correct
- [ ] RETURNED: any document is marked Incorrect
- [ ] REFERRED: verifier triggers task-level escalation (supersedes all document-level outcomes)

**AC-FLC-04 — Save-as-you-go**
Ensures verifier progress is persisted per document independently, allowing work to be paused and resumed without data loss across page reloads or browser sessions.
- [ ] Each document's decision (Correct/Incorrect), all check item outcomes, and remarks are saved independently per document
- [ ] A verifier can save one document and proceed to the next without submitting the full task
- [ ] In-progress state (saved documents) is preserved across browser close and page reload

**AC-FLC-05 — Submission Guardrails**
Prevents accidental or incomplete submissions by blocking the action until all documents carry a decision and requiring explicit verifier confirmation before the task is finalized.
- [ ] The submission button is disabled until all documents have a recorded decision
- [ ] A confirmation dialog is presented to the verifier before final submission is accepted

---

### 2.3 Capability: Async Car Check Integration

**AC-CAR-01 — Car Check Status Visibility**
Confirms the verifier always has a clear, up-to-date view of whether car check data is pending, in-flight, or received, updated automatically on push callback from the provider.
- [ ] The verification UI displays car check status: `none`, `waiting`, or `received`
- [ ] The status updates when a push callback is received from the car check provider

**AC-CAR-02 — Manual Refresh**
Allows a verifier to actively pull the latest car check data on demand without reloading the full task page, reducing wait friction.
- [ ] A verifier can manually trigger a refresh to pull the latest car check data without reloading the task

**AC-CAR-03 — Bypass / Cut-off**
Ensures verifiers are never blocked from completing a task by an unresponsive or slow car check provider — submission is always available regardless of car check status.
- [ ] A verifier can submit a final decision when car check status is `waiting` — the system does not block submission
- [ ] There is no `awaiting_car_check` task state; bypass is always available

**AC-CAR-04 — Late Data Handling**
Defines how the system reacts when car check data arrives after the verifier has already made or is mid-way through a decision, ensuring no new data is silently ignored.
- [ ] If car check data arrives while the task is `IN_PROGRESS`: the task data updates and a "Revised" alert is shown on next load
- [ ] If car check data arrives after the task is `COMPLETED`: the task transitions to `PENDING_REVIEW` and a new `work-entry` is published to Solomon automatically

**AC-CAR-05 — Re-decision Audit**
Verifies that a verifier's revised decision following late car check data is recorded as a new, separate audit event — leaving the original decision intact and traceable.
- [ ] A re-decision following a car check late arrival creates a new `TaskCompletionEvent` with `is_re_decision: true`
- [ ] The original `TaskCompletionEvent` is never mutated

---

### 2.4 Capability: Safety & Integrity Guardrails

**AC-SIG-01 — Dual-Hash Computation**
Establishes the foundation of change detection: a live hash of the document's current state (`context_hash`) and a snapshot hash taken at decision time (`decision_hash`), which are compared to detect any post-decision modification.
- [ ] `context_hash` is computed as SHA-256 of document `data` (jsonb) and `file_urls`, and is recalculated on every data or file URL change
- [ ] `decision_hash` is captured at the moment the verifier submits their decision (snapshot of `context_hash` at decision time)

**AC-SIG-02 — Mid-review Change Detection**
Ensures a verifier is immediately alerted and forced to re-review if document data changes while they are actively working on the task, preventing a decision being submitted against stale data.
- [ ] If `context_hash ≠ decision_hash` while the task is `IN_PROGRESS`, a "Revised" alert is shown identifying the changed documents
- [ ] Pre-filled decisions for changed documents are cleared
- [ ] The submission button is disabled until the verifier has re-reviewed all changed documents

**AC-SIG-03 — Post-completion Change Detection**
Closes the approval leakage window: any document modification after a task is closed automatically triggers PENDING_REVIEW, preventing an approved decision from remaining attached to data the verifier never assessed.
- [ ] If `context_hash` changes after a task reaches `COMPLETED`, the task transitions to `PENDING_REVIEW` automatically
- [ ] A new `work-entry` is published to Solomon
- [ ] Decisions on changed documents are invalidated pending re-review

**AC-SIG-04 — Immutable Audit Trail**
Satisfies regulatory and compliance requirements by guaranteeing every verification decision — including re-decisions — is permanently recorded as an append-only event with full context and is never modified or deleted.
- [ ] Every `COMPLETED` transition creates a new, immutable `TaskCompletionEvent`
- [ ] `TaskCompletionEvent` records: `outcome`, `submitted_by`, `trigger_reason` (`human` or `human_audit`), `callback_status`, `is_re_decision`
- [ ] No `TaskCompletionEvent` is ever updated or deleted after creation
- [ ] Both the original decision and any re-decision are fully visible in the audit trail

---

### 2.5 Capability: Re-flow

**AC-RFL-01 — Re-flow Detection**
Confirms the system correctly identifies a resubmitted application by its surrogate key, creates a new versioned task, and links it to the prior task for a complete audit chain.
- [ ] When a new task is created with a `surrogateKey` matching a previously `COMPLETED` task, a re-flow is triggered
- [ ] The new task is assigned an auto-incremented version number (e.g., v1 → v2)
- [ ] All versions sharing a `surrogateKey` are linked and auditable

**AC-RFL-02 — Smart Result Copy**
Eliminates re-verification of unchanged documents: previous results are automatically copied where content is identical, while only genuinely changed or new documents are surfaced to the verifier for action.
- [ ] Documents whose `context_hash` matches the previous version have their verification result automatically copied and are shown as already-decided to the verifier
- [ ] Documents whose `context_hash` differs from the previous version are marked `is_changed = true` and require verifier re-verification
- [ ] New documents with no previous version are shown as not-yet-verified

**AC-RFL-03 — Traceability**
Ensures every document in a resubmission can be traced back to its counterpart in the previous version, giving auditors and verifiers a clear lineage of what was re-verified versus carried forward.
- [ ] Every document in a re-flow task contains a `previous_document_id` linking to the corresponding document from the prior version
- [ ] Verifier sees only changed and new documents as requiring action; previously verified unchanged documents are pre-resolved

---

## 3. UAT Approach

### 3.1 UAT Entry Criteria

Before UAT begins, the following must be satisfied:

- [ ] All Acceptance Criteria in Section 2 have passed developer testing
- [ ] Test environment with Solomon (QA worklist) integration is available
- [ ] Test environment with Haibara (car check) callback simulation is available
- [ ] A test client system (Onigiri sandbox or test harness) capable of calling `POST /task` is ready
- [ ] Operations team has access to the configuration UI to manage Data Items and Policy Items
- [ ] UAT test cases mapped to each AC are prepared and signed off by QA lead

---

### 3.2 UAT Scenarios

#### Module 1: Task Lifecycle & QA Distribution

| Test ID | Scenario | Steps | Expected Result |
|---------|----------|-------|----------------|
| UAT-UVE-01 | Happy path — full task completion | Create task via POST /task → Verifier opens from Solomon → Marks all docs Correct → Submits | Task reaches COMPLETED; APPROVED webhook sent to client |
| UAT-UVE-02 | RETURNED outcome | Create task → Verifier marks one doc Incorrect → Submits | Task COMPLETED; RETURNED webhook sent; rejection_reason stored |
| UAT-UVE-03 | REFERRED outcome | Create task → Verifier clicks Refer → Submits | Task COMPLETED; REFERRED webhook sent; outcome supersedes document-level decisions |
| UAT-UVE-04 | PENDING_REVIEW via context change | Complete a task → Modify document data in source system → Verify hash recalculated | Task transitions to PENDING_REVIEW; new work-entry in Solomon |
| UAT-UVE-05 | Re-decision audit integrity | Trigger PENDING_REVIEW → Verifier re-decides | Two TaskCompletionEvents exist: original + re-decision with is_re_decision: true; original unchanged |

#### Module 2: Flexible Logic Configuration

| Test ID | Scenario | Steps | Expected Result |
|---------|----------|-------|----------------|
| UAT-FLC-01 | Add new document type without code deploy | Operations adds new document type + Data Items + Policy Items via config UI | New document type appears in verification task with correct checks; no deployment required |
| UAT-FLC-02 | Save-as-you-go resilience | Verifier saves 2 of 3 documents → Closes browser → Reopens task | Task resumes showing 2 saved documents; 1 remaining |
| UAT-FLC-03 | Submission blocked until all docs decided | Open task with 3 docs → Decide only 2 → Attempt submit | Submit button disabled; system shows remaining undecided documents |
| UAT-FLC-04 | Confirmation dialog on submit | All docs decided → Click Submit | Confirmation dialog appears; task only submits after explicit confirmation |

#### Module 3: Async Car Check Integration

| Test ID | Scenario | Steps | Expected Result |
|---------|----------|-------|----------------|
| UAT-CAR-01 | Car check arrives before decision | Create task → Simulate car check callback → Verifier opens task | Car check status shows `received`; data visible to verifier |
| UAT-CAR-02 | Bypass when car check is waiting | Create task → Do NOT send car check callback → Verifier submits decision | Task completes without car check; webhook sent to client |
| UAT-CAR-03 | Late car check after COMPLETED | Complete task → Send car check callback | Task transitions to PENDING_REVIEW; new work-entry in Solomon |
| UAT-CAR-04 | Manual refresh | Task in waiting state → Verifier clicks Refresh | System re-checks for car check data without full page reload |

#### Module 4: Safety & Integrity Guardrails

| Test ID | Scenario | Steps | Expected Result |
|---------|----------|-------|----------------|
| UAT-SIG-01 | Mid-review data change detected | Verifier opens task → Source system updates document data mid-review → Verifier continues | "Revised" alert shown; pre-filled decisions cleared for changed docs; Submit disabled |
| UAT-SIG-02 | Post-completion hash change | Complete task → Source system updates document data | Task transitions to PENDING_REVIEW automatically; new Solomon work-entry created |
| UAT-SIG-03 | Audit trail completeness | Complete a task, trigger PENDING_REVIEW, re-decide | Both TaskCompletionEvents visible in audit log; original unmodified; re-decision flagged |

#### Module 5: Re-flow

| Test ID | Scenario | Steps | Expected Result |
|---------|----------|-------|----------------|
| UAT-RFL-01 | Re-flow with unchanged docs | Complete task v1 → Resubmit identical docs with same surrogateKey | v2 task created; all docs shown as already-verified (results copied); verifier has no action |
| UAT-RFL-02 | Re-flow with partially changed docs | Complete task v1 → Resubmit with 1 doc changed → Open v2 task | Changed doc marked is_changed = true; requires re-verification; other docs pre-resolved |
| UAT-RFL-03 | Version traceability | Inspect v2 documents | Each document has previous_document_id linking to v1 counterpart; all versions linked via surrogateKey |
| UAT-RFL-04 | New document in re-flow | Complete task v1 → Resubmit with 1 new doc added | New doc shown as not-yet-verified; existing unchanged docs pre-resolved |

---

### 3.3 UAT Roles & Responsibilities

| Role | Responsibility |
|------|---------------|
| **UAT Lead (Operations)** | Own sign-off; coordinate verifier testers; escalate blockers |
| **QA Verifier Testers (2–3 staff)** | Execute Module 1–4 scenarios as end users in the QA worklist (Solomon) |
| **Operations Config Tester** | Execute Module 2 configuration scenarios (adding document types, rules) |
| **Engineering Support** | Provide test environment access; simulate Haibara callbacks; fix defects |
| **Product Owner** | Triage defect severity; accept or reject UAT completion |

---

### 3.4 Defect Classification

| Severity | Definition | UAT Impact |
|----------|-----------|------------|
| **P1 — Blocker** | Acceptance Criterion fails; core business function broken | Blocks UAT sign-off |
| **P2 — Major** | Incorrect behaviour but workaround exists | Must be resolved before sign-off |
| **P3 — Minor** | UI/UX issue; does not affect data integrity or outcomes | May be deferred to post-launch fix |

---

### 3.5 UAT Exit Criteria

UAT is considered complete and Phase 1 is ready for production release when:

- [ ] All P1 and P2 defects are resolved and re-tested
- [ ] 100% of UAT scenarios in Section 3.2 have been executed and passed
- [ ] Operations team confirms the configuration UI meets operational needs
- [ ] UAT Lead sign-off obtained
- [ ] All Phase 1 Acceptance Criteria (Section 2) formally accepted

---

*AI-First Verification (Wasabi routing, auto-complete, spot-check sampling) will be addressed in the Phase 2 BRD.*
