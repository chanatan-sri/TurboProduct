# Research: Loan Restructure — Pre-Approval Capability Concept

**Author**: Product Owner session — 2026-03-10
**Branch**: restructure-1.3
**Status**: Implemented — CAPABILITY.md, feature files, and changelog created. Research updated to reflect all design decisions made during session.

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
| EasyPass authority matching | EasyPass flag returned by pre-built = `true` means the campaign's assessed risk level falls within local CO authority. A CO initiating a pre-approval for an EasyPass campaign already satisfies the approval authority requirement — no Approval Request is generated. Pre-approval auto-converts to Draft on plan selection. |
| Non-EasyPass authority gap | EasyPass flag = `false` means the campaign's risk level exceeds local CO authority. Approval Request is required — submitted to the designated approver above local level. |
| EasyPass is not a shortcut | EasyPass is a statement of authority alignment. The CO is not bypassing approval — the CO IS the approval authority for that risk level. |
| EasyPass source | The EasyPass flag is returned by Campaign Eligibility Pre-Build at evaluation time. It is **not** stored on the campaign configuration. Campaign configuration only stores eligibility and risk criteria — pre-built evaluates them against the customer context and returns the flag. |

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
| `converted` | Draft Initializer used this pre-approval to create an application. **Not terminal — reusable.** The same pre-approval can generate additional Drafts as long as it is not `rejected` or `expired` (e.g., if a prior Draft was withdrawn or rejected). |

### Path by EasyPass

```
EasyPass (local authority met):
  plan selected → created → converted  [auto — no explicit CO convert action]
  No approval step. No expiry. Draft Initializer triggered automatically on plan selection.

Non-EasyPass (authority gap — escalation required):
  plan selected → created → pending_approval → approved (carries expiry) → converted (reusable)
                                             → rejected                    ↑ can convert again
                                             → expired
  CO converts via BOS (Customer Detail) or CO Worklist.
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

## 6. Entry Points

Two systems navigate into pre-approval. Each has a defined scope:

| Entry Point | Allowed Actions | Pre-conditions |
|---|---|---|
| **BOS** → Customer List → Customer Detail | **Create new pre-approval** — triggers master data fetch from DaVinci + Campaign Eligibility Pre-Build; opens pre-approval screen. Also allows **accessing an existing approved pre-approval** to convert to Draft. | Master data fetch must succeed; pre-built must return ≥1 eligible campaign (creation path). |
| **CO Worklist** | **Access approved pre-approval only** — navigates CO to the approved record for conversion to Draft. Cannot create a new pre-approval. | Pre-approval must be in `approved` state. |

### Pre-Screen Sequence (BOS — creation path)

Before the pre-approval screen opens, BOS must complete these two calls in sequence:

1. **Master Data Fetch** — Retrieve from DaVinci (Customer & Product Master Data):
   - Person detail (customer profile, identification)
   - Contract detail (existing loan reference, outstanding balance, loan status, DPD, prior restructure count, loan age)
   - Collateral data (collateral type, valuation, status)

2. **Campaign Eligibility Pre-Build** — Called with customer context (including contract and collateral data from step 1). Returns eligible campaigns + EasyPass flag per campaign.

If either call fails or pre-built returns zero eligible campaigns, the pre-approval screen must not open. A snapshot of the fetched master data is stored on the pre-approval record at creation time.

---

## 7. Feature Inventory

| Feature | Description |
|---|---|
| **Pre-Approval Request Creation** | Entry: BOS only. BOS calls master data fetch from DaVinci, then Campaign Eligibility Pre-Build, before screen opens. Dedicated pre-approval screen (not Smart Form) displays eligible campaigns (name, min/max tenor, EasyPass flag) with client-side EasyPass and tenor filters. CO selects a plan. Pre-approval record created with master data snapshot stored. EasyPass: auto-converts to Draft. Non-EasyPass: persists in `created`. |
| **Approval Request** | Non-EasyPass path only. Pre-approval submitted to designated approver. Outcomes: Approve or Reject. Approver may revise own decision before `converted`. CO may not. |
| **Pre-Approval Expiry Management** | Non-EasyPass path only. Approved pre-approval carries an expiry date. System-configured default. Approver may override at approval time. System auto-transitions `approved` → `expired`. Secondary guard at conversion time. EasyPass carries no expiry. |
| **Draft Initializer** | Converts a valid pre-approval into a Draft. Entry: BOS or CO Worklist (non-EasyPass); auto-triggered for EasyPass. Pre-populates form with selected plan, existing loan reference, and master data snapshot. Sets `easypass_flag`, `pre_approval_id`, `pre_approval_snapshot` on application record. Pre-approval remains reusable after conversion. |

---

## 8. Application Record Additions

### Pre-Approval Record Fields (stored at creation time)

| Field | Type | Source | Purpose |
|---|---|---|---|
| `customer_reference` | reference | BOS context | Links pre-approval to the customer |
| `selected_campaign` | reference | CO plan selection | Campaign chosen from pre-built results |
| `easypass_flag` | boolean | Pre-built output | Determines approval path |
| `reason_for_restructure` | enum | CO dropdown selection (required) | Reason type — surfaced to approver |
| `reason_detail` | string | CO free-text (optional) | Additional explanation — surfaced to approver |
| `supporting_documents` | reference[] | CO file upload (optional) | Document references (e.g. medical cert, income proof) — accessible by approver |
| `financial_snapshot` | JSON | CO-reviewed (pre-filled from DaVinci, editable) | Primary income, supplementary income, monthly debt burden, debt end date, monthly expenses, tax due date — surfaced to approver in Approval Request view |

### Application Record Fields (set by Draft Initializer at conversion)

| Field | Type | Set by | Purpose |
|---|---|---|---|
| `easypass_flag` | boolean | Draft Initializer (from pre-built) | Drives approval routing at `pending_approval` in the full workflow |
| `pre_approval_id` | reference | Draft Initializer | Links application to originating pre-approval for audit trail and change detection |
| `pre_approval_snapshot` | JSON | Draft Initializer | Point-in-time copy of pre-approval data (including master data snapshot and reason for restructure) — compared at Draft submission for change detection |

Note: There is no `restructure_flag` on the application record. The campaign's `campaign_type = restructure` implicitly identifies the application type. The `easypass_flag` on the application record is set by the Draft Initializer based on the value returned by pre-built — not from the campaign configuration.

---

## 9. Changes to Existing Capabilities

### 8a. Smart Form — No Change

Pre-approval plan selection is a **dedicated screen**, not a Smart Form. Smart Form is used for the Draft application form after conversion. Dynamic Field Options (previously proposed) is not needed and has been removed.

The dedicated pre-approval screen renders eligible campaigns from the pre-built response directly, with client-side filtering by:
- **EasyPass** — All / EasyPass only / Non-EasyPass only
- **Tenor** — Numeric range input; filters campaigns whose min–max tenor overlaps the CO's input

Each campaign card displays: campaign name, min/max tenor, EasyPass indicator.

### 8b. Loan Campaign Configuration — Additive Changes

#### Add `restructure` to Campaign Types table

Add one new row to the existing `### Campaign Types` table in the CAPABILITY.md:

| Type | Display Name | Description |
|---|---|---|
| `restructure` | Restructure | Loan restructure offer for an existing active borrower. A new Onigiri application is created using the campaign's Application Template, pre-populated with the existing loan reference. Uses 5 configuration dimensions. Participates in Campaign Eligibility Pre-Build — Pre-Build evaluates Eligibility Criteria including restructure-specific criteria (existing loan DPD, loan status, prior restructure count, loan age, outstanding balance) and Risk Strategy. |

### 8c. Underwriting Workflow — Additive Changes

**1. Register Topology D and Topology E**

| Topology | Entity | States |
|---|---|---|
| D — Pre-Approval | Pre-approval request | `created`, `pending_approval`, `approved`, `rejected`, `expired`, `converted` |
| E — Restructure Application | Restructure loan application (end-to-end) | Combined view: Topology D (pre-approval) → Topology A (underwriting) with EasyPass routing and change detection |

**2. New state: `pre_approval`**

Added to Topology D. Runs on the same workflow engine. Pre-HWM — does not write to `state_high_water_mark`. HWM begins at `draft` as in Topology A.

**3. Topology E Entry Flow Diagram**

Topology E uses a flowchart design (not stateDiagram-v2) to show entry points explicitly. Two entry boxes — BOS and CO Worklist — lead into the pre-approval journey:

- **BOS → Customer Detail → Check Pre-Approval**:
  - No approved pre-approval → Creation `[created]`
    - EasyPass: auto-converts to `[converted]`
    - Non-EasyPass: submits for Approval → Pending Approval `[pending_approval]` → Result → Confirmation `[approved]` or Rejected `[rejected]`
  - Approved pre-approval exists → Confirmation `[approved]`
- **CO Worklist** → Confirmation `[approved]` directly (access-only — cannot create)
- From Confirmation: CO converts to Draft `[converted]` (reusable); or system auto-expires `[expired]`

**4. EasyPass routing at `pending_approval` (Topology A — Restructure applications)**

At `pending_approval` state entry for restructure applications, a configurable execution step evaluates `easypass_flag` on the application record and sets the approver queue:

| `easypass_flag` | Approval Route |
|---|---|
| `true` | Local approver — CO authority already satisfies the risk level by design (EasyPass campaigns only produce local-authority risk) |
| `false` | Standard escalation path (designated higher authority) |

**Note**: There is no MEDIUM/HIGH override for EasyPass. EasyPass by design only designates campaigns whose risk level falls within local authority. A campaign that could produce MEDIUM/HIGH risk would not be designated EasyPass by pre-built.

**5. Change detection execution step at Draft submission (Non-EasyPass restructure applications only)**

At Draft submission, for applications with a `pre_approval_id` and `easypass_flag = false`:

| Condition | Outcome |
|---|---|
| Draft data matches `pre_approval_snapshot` (no changes) | Skip `pending_approval` — approver already reviewed at pre-approval stage |
| Delta detected (data changed since pre-approval) | Route through `pending_approval` as normal |
| No `pre_approval_id` (application created without pre-approval) | Full workflow — `pending_approval` runs unchanged |

---

## 10. Full Workflow Paths for Restructure

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

## 11. Files to Create / Modify

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
| `capabilities/smart-form/CAPABILITY.md` | No change — Dynamic Field Options removed; pre-approval uses a dedicated screen, not Smart Form |
| `capabilities/loan-campaign-configuration/CAPABILITY.md` | Add `restructure` row to Campaign Types table |
| `capabilities/underwriting-workflow/CAPABILITY.md` | Register Topology D; add `pre_approval` state; add EasyPass routing rule at `pending_approval`; add change detection execution step |

---

## 12. Open Dependencies (Not in Scope for 1.3)

| Dependency | Status | Detail |
|---|---|---|
| `pre-built` capability | Open — external | Provides campaign eligibility results consumed by Pre-Approval Request Creation. Exists separately — not defined here. Interface: returns eligible campaigns for a given customer + loan context + EasyPass flag per campaign. |
| DaVinci (Customer & Product Master Data) | **Resolved** | Source of person detail, contract detail, and collateral data. Called by BOS before pre-approval screen opens. Data used as context for CO and passed to pre-built as input. |
| Sensei notification | Deferred | When Approval Request is submitted, approver notification via Sensei TaskCreationRequest. Deferred — not in scope for 1.3. |
| CO Worklist implementation | Open — Sensei/BOS | CO Worklist must surface approved pre-approvals as actionable items. Implementation owner (Sensei or BOS) not yet resolved. |
