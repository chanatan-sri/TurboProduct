# Research: Loan Restructure — Pre-Approval Capability Concept

**Author**: Product Owner session — 2026-03-10
**Branch**: restructure-1.3
**Status**: Concept confirmed — ready for CAPABILITY.md authoring

---

## 1. Layer Classification

| Question | Answer |
|---|---|
| Which layer? | **@CAPABILITY** (new) + amendments to three existing capabilities + one product-level change |
| Which product? | **Onigiri** — Loan Origination System |
| Existing overlap? | None. Pre-approval is a new intake path not present in any existing capability. |
| New product or new capability? | New capability within Onigiri — restructure is a new front door to the same post-Draft workflow. If restructure develops a distinct post-disbursement lifecycle (servicing, settlement), re-evaluate at that point. |

---

## 2. Problem Being Solved

Today, the CO submits a full application and waits for the approver before knowing whether a plan is viable. The offer is made blind — the CO has no way to know upfront which plans a customer can get or what the approval path will be.

The pre-approval capability gives the CO certainty about what a customer can get — and what approval path each option will follow — before creating a formal application.

| Without pre-approval | With pre-approval |
|---|---|
| CO guesses which plan might work | CO sees which campaigns the customer qualifies for before committing |
| Customer waits before knowing if the offer is real | CO makes a grounded offer at the branch on the spot |
| CO cannot tell customer expected timeline | CO can tell customer: this plan is within my authority (EasyPass), that plan needs a higher approver |

---

## 3. Scope Boundary

**In scope for restructure 1.3:**
- New capability: `pre-approval`
- New feature in `smart-form`: Dynamic Field Options
- Campaign configuration amendments: add `restructure` campaign type
- Underwriting Workflow amendments: Topology D registration, `pre_approval` state, EasyPass routing at `pending_approval`, change detection execution step

**Explicitly out of scope for restructure 1.3:**
- `pre-built` capability (campaign eligibility screener) — exists separately, referenced as dependency
- Sensei notification for Approval Request — not touched in this scope
- Existing loan data source resolution (Core Banking vs Genesis) — open dependency, not resolved here

---

## 4. New Capability: `pre-approval`

### Business Function

Enable a CO to evaluate which restructure plans a customer is eligible for and present viable options before initiating a formal application — and route the application to the correct approval authority based on campaign EasyPass designation.

### Why It Exists (First Principles)

- **Offer accuracy**: CO needs to know what is achievable before making a commitment to the customer. Without this, offers are speculative and the CO risks presenting plans that will not be approved.
- **Authority clarity**: Not all restructure campaigns require escalation to a higher approver. EasyPass campaigns fall within local CO authority — the CO already has the authority to proceed without waiting. This must be surfaced before the application is created, not discovered at `pending_approval`.
- **Queue efficiency**: Only applications that genuinely require a higher authority should enter the approver queue. Pre-approval filters noise before it becomes workflow volume.

### EasyPass Business Rule

| Rule | Detail |
|---|---|
| EasyPass authority matching | `easypass_enabled = true` means the campaign's assessed risk level falls within local CO authority. A CO initiating a pre-approval for an EasyPass campaign already satisfies the approval authority requirement — no Approval Request is generated. |
| Non-EasyPass authority gap | `easypass_enabled = false` means the campaign's risk level exceeds local CO authority. Approval Request is required — submitted to the designated approver above local level. |
| EasyPass is not a shortcut | EasyPass is a statement of authority alignment. The CO is not bypassing approval — the CO IS the approval authority for that risk level. |

---

## 5. Pre-Approval Lifecycle (Topology D)

Pre-approval runs as **Topology D** on the Underwriting Workflow engine — same audit trail, same execution step infrastructure, same transition atomicity as Topologies A, B, and C.

### State Table

| State | Description |
|---|---|
| `created` | CO initiated pre-approval request. Not yet submitted. Stays here if CO closes without submitting. |
| `pending_approval` | Submitted for Approval Request. Waiting for designated approver. Non-EasyPass path only. |
| `approved` | Approver confirmed. Valid until expiry date. Non-EasyPass path only. |
| `rejected` | Approver rejected. CO must initiate a new pre-approval request. Non-EasyPass path only. |
| `expired` | Approved pre-approval passed its expiry date. Invalidated — new request required. Non-EasyPass path only. |
| `converted` | Draft Initializer used this pre-approval to create an application. Terminal state. |

### Path by EasyPass

```
EasyPass (local authority met):
  created → converted
  No approval step. No expiry. CO converts directly to Draft.

Non-EasyPass (authority gap — escalation required):
  created → pending_approval → approved (carries expiry) → converted
                             → rejected
                             → expired
```

### Expiry Rule

| Path | Expiry |
|---|---|
| EasyPass | No expiry — EasyPass pre-approval carries no expiry date |
| Non-EasyPass | Carries expiry date. Default duration set by system configuration. Approver may override expiry date at time of approval. On expiry: transitions to `expired` — CO must submit a new pre-approval request. |

**Why expiry exists**: An approved pre-approval is a snapshot of campaign terms, eligibility conditions, and risk assessment at a point in time. If any of these change (campaign archived, eligibility rules updated, risk strategy revised), the pre-approval may no longer reflect valid conditions. Expiry enforces a recheck cadence.

### Approval Decision Rules

| Actor | Can do |
|---|---|
| Approver | Approve or Reject. May revise own previous decision (e.g., change Approve → Reject) as long as pre-approval has not reached `converted`. |
| CO | Cannot influence or change the approver's decision after submission. |

---

## 6. Feature Inventory

| Feature | Description |
|---|---|
| **Pre-Approval Request Creation** | CO initiates a pre-approval request for a selected plan + customer. Plan options are sourced from pre-built (external dependency). Plan selection uses Smart Form with Dynamic Field Options feature. |
| **Approval Request** | Non-EasyPass path only. Pre-approval submitted to designated approver. Outcomes: Approve or Reject. Approver may revise own decision. CO may not. |
| **Pre-Approval Expiry Management** | Non-EasyPass path only. Approved pre-approval carries an expiry date. System-configured default. Approver may override at approval time. On expiry: pre-approval invalidated, new request required. |
| **Draft Initializer** | Converts a valid pre-approval (`approved` or EasyPass `created`) into a Draft application. Pre-populates form with selected plan data and existing loan reference. Stores pre-approval snapshot on application record for downstream change detection. Sets `easypass_flag` on application record. |

---

## 7. Application Record Additions

| Field | Type | Set by | Purpose |
|---|---|---|---|
| `easypass_flag` | boolean | Draft Initializer (from campaign `easypass_enabled`) | Drives approval routing at `pending_approval` in the full workflow |
| `pre_approval_id` | reference | Draft Initializer | Links application to originating pre-approval for audit trail and change detection |
| `pre_approval_snapshot` | JSON | Draft Initializer | Point-in-time copy of pre-approval data for change detection at Draft submission |

Note: There is no `restructure_flag` on the application record. The campaign's `campaign_type = restructure` implicitly identifies the application type. The `easypass_flag` on the application record is set by the Draft Initializer based on the value returned by pre-built — not from the campaign configuration.

---

## 8. Changes to Existing Capabilities

### 8a. Smart Form — New Feature

**Dynamic Field Options**: Enables a `select` field to bind its option list to an external service response at render time, rather than static configured values. Required by Pre-Approval Request Creation to surface eligible plan options from pre-built.

New field definition property:

| Property | Description |
|---|---|
| `options_source` | (Optional) External service endpoint that returns the option list at render time. When present, overrides any static `options` configuration. |

### 8b. Loan Campaign Configuration — Additive Changes

#### Add `restructure` to Campaign Types table

Add one new row to the existing `### Campaign Types` table in the CAPABILITY.md:

| Type | Display Name | Description |
|---|---|---|
| `restructure` | Restructure | Loan restructure offer for an existing active borrower. A new Onigiri application is created using the campaign's Application Template, pre-populated with the existing loan reference. Uses 5 configuration dimensions. Participates in Campaign Eligibility Pre-Build — Pre-Build evaluates Eligibility Criteria including restructure-specific criteria (existing loan DPD, loan status, prior restructure count, loan age, outstanding balance) and Risk Strategy. |

### 8c. Underwriting Workflow — Additive Changes

**1. Register Topology D**

| Topology | Entity | States |
|---|---|---|
| D — Pre-Approval | Pre-approval request | `created`, `pending_approval`, `approved`, `rejected`, `expired`, `converted` |

**2. New state: `pre_approval`**

Added to Topology D. Runs on the same workflow engine. Pre-HWM — does not write to `state_high_water_mark`. HWM begins at `draft` as in Topology A.

**3. EasyPass routing at `pending_approval` (Topology A — Restructure applications)**

At `pending_approval` state entry for restructure applications, a configurable execution step evaluates `easypass_flag` on the application record and sets the approver queue:

| `easypass_flag` | Risk Level | Approval Route |
|---|---|---|
| `true` | LOW | Local approver (CO authority satisfied) |
| `true` | MEDIUM / HIGH | Escalate — risk level overrides EasyPass |
| `false` | Any | Standard escalation path (CA or higher) |

**4. Change detection execution step at Draft submission (Non-EasyPass restructure applications only)**

At Draft submission, for applications with a `pre_approval_id` and `easypass_flag = false`:

| Condition | Outcome |
|---|---|
| Draft data matches `pre_approval_snapshot` (no changes) | Skip `pending_approval` — approver already reviewed at pre-approval stage |
| Delta detected (data changed since pre-approval) | Route through `pending_approval` as normal |
| No `pre_approval_id` (application created without pre-approval) | Full workflow — `pending_approval` runs unchanged |

---

## 9. Full Workflow Paths for Restructure

### Path A — EasyPass, via pre-approval
```
Pre-Approval (created → converted)
→ Draft → Risk Assessment → pending_approval (local routing)
→ Create Facility → ... → Funded
```

### Path B — Non-EasyPass, via pre-approval (no data change)
```
Pre-Approval (created → pending_approval → approved → converted)
→ Draft → [change detection: clean] → skip pending_approval
→ Create Facility → ... → Funded
```

### Path C — Non-EasyPass, via pre-approval (data changed at Draft)
```
Pre-Approval (created → pending_approval → approved → converted)
→ Draft → [change detection: delta] → pending_approval (CA routing)
→ Create Facility → ... → Funded
```

### Path D — No pre-approval (direct restructure application)
```
Draft → Risk Assessment → pending_approval (full routing, unchanged)
→ Create Facility → ... → Funded
```

---

## 10. Files to Create / Modify

### Files to CREATE

| File | Layer | Purpose |
|---|---|---|
| `capabilities/pre-approval/CAPABILITY.md` | Capability | Full capability definition: business function, feature inventory, business rules, state machine, EasyPass routing table, NFRs, open questions |
| `capabilities/pre-approval/features/FEATURE_pre-approval-request-creation.md` | Feature | Pre-approval initiation feature spec |
| `capabilities/pre-approval/features/FEATURE_approval-request.md` | Feature | Non-EasyPass approval flow feature spec |
| `capabilities/pre-approval/features/FEATURE_pre-approval-expiry-management.md` | Feature | Expiry management feature spec |
| `capabilities/pre-approval/features/FEATURE_draft-initializer.md` | Feature | Draft creation from pre-approval feature spec |
| `changelogs/CHANGELOG_005_pre-approval-capability.md` | Changelog | Records all changes in this session |

### Files to MODIFY

| File | Change |
|---|---|
| `PRODUCT.md` | Add pre-approval to capability registry; update user flow with Topology D paths; update Product Boundary IS section |
| `capabilities/smart-form/CAPABILITY.md` | Add Dynamic Field Options to feature inventory; add `options_source` to field definition properties |
| `capabilities/loan-campaign-configuration/CAPABILITY.md` | Add `restructure` row to Campaign Types table |
| `capabilities/underwriting-workflow/CAPABILITY.md` | Register Topology D; add `pre_approval` state; add EasyPass routing rule at `pending_approval`; add change detection execution step |

---

## 11. Open Dependencies (Not in Scope for 1.3)

| Dependency | Detail |
|---|---|
| `pre-built` capability | Provides campaign eligibility results consumed by Pre-Approval Request Creation. Exists separately — not defined here. Interface: returns eligible campaigns for a given customer + loan context. |
| Existing loan data source | Pre-approval check may need existing loan status (DPD, balance, prior restructure count). Candidate sources: Core Banking (live) and/or Genesis (Contract & Collateral Master Data). Resolution deferred. |
| Sensei notification | When Approval Request is submitted, approver notification via Sensei TaskCreationRequest. Deferred — not in scope for 1.3. |
