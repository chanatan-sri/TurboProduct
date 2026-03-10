# Capability: Pre-Approval

**Product**: Onigiri — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Credit
**Product Owner**: TBD (Credit PO)
**Status**: 📝 Draft
**Last Updated**: 2026-03-10

---

## Business Function

Enable a CO to evaluate which restructure plans a customer is eligible for and present viable options before initiating a formal application — and route the resulting application to the correct approval authority based on the EasyPass designation returned by Campaign Eligibility Pre-Build.

## Why It Exists (First Principles)

- **Offer accuracy**: CO needs to know what is achievable before making a commitment to the customer. Without pre-approval, offers are speculative — the CO risks presenting plans that will not be approved.
- **Authority clarity**: Not all restructure campaigns require escalation to a higher approver. EasyPass campaigns fall within local CO authority — the CO already has the authority to proceed without waiting. This must be surfaced before the application is created, not discovered at `pending_approval`.
- **Queue efficiency**: Only applications that genuinely require a higher authority should enter the approver queue. Pre-approval filters noise before it becomes workflow volume.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| [Pre-Approval Request Creation](features/FEATURE_pre-approval-request-creation.md) | Concept | CO arrives at pre-approval screen from BOS Customer Detail. Eligible campaigns are already loaded — Pre-Build was called by BOS before the screen opened. CO selects a plan. Pre-approval record created on plan selection. |
| [Approval Request](features/FEATURE_approval-request.md) | Concept | Non-EasyPass path only. Pre-approval submitted to designated approver. Outcomes: Approve or Reject. Approver may revise own decision. CO may not. |
| [Pre-Approval Expiry Management](features/FEATURE_pre-approval-expiry-management.md) | Concept | Non-EasyPass path only. Approved pre-approval carries an expiry date. System-configured default. Approver may override at approval time. Expired pre-approval is invalidated — new request required. |
| [Draft Initializer](features/FEATURE_draft-initializer.md) | Concept | Converts a valid pre-approval into a Draft application. Pre-populates form with selected plan data and existing loan reference. Stores pre-approval snapshot on application record. Sets `easypass_flag` from pre-built output. Terminal action for this capability — full workflow continues from Draft. |

---

## Business Rules

### EasyPass Authority Rule

EasyPass is a statement of authority alignment — not a shortcut. A CO initiating a pre-approval for an EasyPass campaign already satisfies the approval authority requirement because the campaign's risk level falls within local CO authority. The CO is not bypassing approval — the CO IS the approval authority for that risk level.

| EasyPass (from pre-built) | Meaning | Pre-Approval Behaviour |
|---|---|---|
| `true` | Campaign risk level is within local CO authority | No Approval Request generated — CO converts directly to Draft. No expiry on pre-approval. |
| `false` | Campaign risk level exceeds local CO authority | Approval Request required — submitted to designated approver. Approved pre-approval carries expiry date. |

### Pre-Approval Lifecycle (Topology D)

Pre-approval runs as **Topology D** on the Underwriting Workflow engine — same audit trail, same execution step infrastructure, same transition atomicity as all other topologies.

| State | Description |
|---|---|
| `created` | Pre-approval record created after CO selects plan from pre-built results. For EasyPass plans, this state is transient — auto-converts to Draft immediately. For non-EasyPass plans, stays here until CO submits Approval Request; persists if CO closes without submitting. |
| `pending_approval` | Submitted for Approval Request. Waiting for designated approver. Non-EasyPass path only. |
| `approved` | Approver confirmed. Valid until expiry date. Non-EasyPass path only. |
| `rejected` | Approver rejected. CO must initiate a new pre-approval request. Non-EasyPass path only. |
| `expired` | Approved pre-approval passed its expiry date — invalidated. New request required. Non-EasyPass path only. |
| `converted` | Draft Initializer used this pre-approval to create an application. Pre-approval remains reusable — additional Drafts can be created from the same pre-approval as long as it is still valid (not `rejected` or `expired`). |

**HWM note**: Pre-approval is pre-Origination. It does not write to `state_high_water_mark`. HWM begins at `draft` as in Topology A.

### Path by EasyPass

```
EasyPass (local authority met):
  plan selected → created → converted → Draft application created
                            [auto — no explicit CO convert action]

Non-EasyPass (authority gap — escalation required):
  plan selected → created → pending_approval → approved → converted (reusable) → Draft application created
                                              → rejected       ↑ can convert again while approved and not expired
                                              → expired
```

### Approval Decision Rules

| Actor | Allowed Actions |
|---|---|
| Approver | Approve or Reject. May revise own previous decision (e.g., Approve → Reject) as long as pre-approval has not reached `converted`. |
| CO | Cannot influence or change the approver's decision after submission. |

### Expiry Rules

| Path | Expiry |
|---|---|
| EasyPass | No expiry — EasyPass pre-approval carries no expiry date. |
| Non-EasyPass | Carries expiry date. Default duration set by system configuration. Approver may override expiry date at time of approval. On expiry: transitions to `expired` — CO must submit a new pre-approval request. |

**Why expiry exists**: An approved pre-approval is a snapshot of campaign terms, eligibility conditions, and risk assessment at a point in time. If any of these change (campaign archived, eligibility rules updated, risk strategy revised), the pre-approval may no longer reflect valid conditions. Expiry enforces a recheck cadence.

### Change Detection at Draft Submission

Applies to **non-EasyPass restructure applications only** — where a higher-authority approver reviewed the plan at pre-approval stage. Executed as a configurable execution step inside the Underwriting Workflow at Draft submission.

| Condition | Outcome at Draft Submission |
|---|---|
| `pre_approval_id` present + `easypass_flag = false` + no data change from snapshot | Skip `pending_approval` — approver already reviewed at pre-approval stage |
| `pre_approval_id` present + `easypass_flag = false` + delta detected | Route through `pending_approval` as normal |
| No `pre_approval_id` (direct restructure application, no pre-approval) | Full workflow — `pending_approval` runs unchanged |

### Application Record Fields Set by Draft Initializer

| Field | Type | Source | Purpose |
|---|---|---|---|
| `easypass_flag` | boolean | Pre-built output (via Draft Initializer) | Drives approval routing at `pending_approval` in full workflow |
| `pre_approval_id` | reference | Draft Initializer | Links application to originating pre-approval for audit trail and change detection |
| `pre_approval_snapshot` | JSON | Draft Initializer | Point-in-time copy of pre-approval data — compared at Draft submission for change detection |

---

## User Flows

### Entry Point

Pre-approval is entered from BOS → Customer List → Customer Detail. Before the pre-approval screen opens, BOS calls Campaign Eligibility Pre-Build with the customer context. The CO arrives on the pre-approval screen with eligible campaigns already loaded — no loading step inside pre-approval itself.

### EasyPass Path

```mermaid
flowchart TD
    A[BOS Customer Detail\nPre-Build already called\neligible campaigns loaded] --> B[CO selects EasyPass campaign]
    B --> C[Pre-approval record created\nstate: created]
    C --> D[Auto-converts to Draft\nDraft Initializer pre-populates form\neasypass_flag = true\nstate: converted]
    D --> E[Draft application created\n→ continues in Underwriting Workflow]
```

### Non-EasyPass Path

```mermaid
flowchart TD
    A[BOS Customer Detail\nPre-Build already called\neligible campaigns loaded] --> B[CO selects non-EasyPass campaign]
    B --> C[Pre-approval record created\nstate: created]
    C --> D[CO submits Approval Request\nstate: pending_approval]
    D --> E{Approver Decision}
    E -- Reject --> F[state: rejected\nCO must start new request]
    E -- Approve --> G[state: approved\nExpiry date set]
    G --> H{Converted\nbefore expiry?}
    H -- No --> I[state: expired\nCO must start new request]
    H -- Yes --> J[Draft Initializer pre-populates form\neasypass_flag = false\npre_approval_snapshot stored\nstate: converted]
    J --> K[Draft application created\n→ continues in Underwriting Workflow]
```

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Topology D on shared engine | Pre-approval runs on the Underwriting Workflow engine — no separate state machine infrastructure |
| Audit trail | Every pre-approval state transition logged with actor, timestamp, and reason — same as all other topologies |
| Snapshot integrity | `pre_approval_snapshot` stored at Draft creation time must be immutable — no post-creation modification |
| Expiry enforcement | System must automatically transition `approved` pre-approvals to `expired` on the expiry date — no manual intervention required |
| Pre-HWM | Pre-approval transitions must never write to `state_high_water_mark` on the linked application record |

---

## Dependencies

| Dependency | Type | Detail |
|---|---|---|
| BOS (Branch Operations System) | Entry point | Pre-approval is initiated from BOS Customer Detail. BOS calls Campaign Eligibility Pre-Build before opening the pre-approval screen. Customer and existing loan context are provided by BOS. |
| Campaign Eligibility Pre-Build | External capability | Called by BOS before the pre-approval screen opens. Returns eligible restructure campaigns + EasyPass flag per campaign. Not defined in this capability. |
| Smart Form — Dynamic Field Options | Feature dependency | Required to render plan selection options sourced dynamically from pre-built. Must be available before Pre-Approval Request Creation can be built. |
| Existing loan data source | Open | Pre-Build needs existing loan status (DPD, balance, prior restructure count). Source — Core Banking and/or Genesis — not yet resolved. Out of scope for 1.3. |
| Sensei notification | Deferred | Approver notification via Sensei TaskCreationRequest when Approval Request is submitted. Out of scope for restructure 1.3. |

---

## Open Questions

- What is the system-configured default expiry duration for approved non-EasyPass pre-approvals?
- Who is the designated approver for non-EasyPass Approval Requests — fixed role or campaign-configurable?
- Can a CO have multiple in-flight pre-approvals for the same customer simultaneously, or is one at a time enforced?
- If pre-built returns no eligible campaigns, what does the CO see and what action is available?
