# CHANGELOG_005 — Pre-Approval Capability

**Date**: 2026-03-10
**Branch**: restructure-1.3
**Layer affected**: Capability (new) · Feature (new) · Capability (modified) × 3

---

## What Changed

### New: Pre-Approval Capability

**File**: `capabilities/pre-approval/CAPABILITY.md`

New Capability added to Onigiri — enables a CO to evaluate which restructure plans a customer is eligible for and route the application to the correct approval authority before creating a formal application.

Key design decisions:
- Runs as **Topology D** on the existing Underwriting Workflow engine — no new state machine infrastructure.
- **EasyPass path**: campaign risk level falls within local CO authority → pre-approval auto-converts to Draft on plan selection. No Approval Request. No expiry.
- **Non-EasyPass path**: campaign risk level exceeds local CO authority → Approval Request submitted to designated approver. Approved pre-approval carries a system-configured expiry date (approver-overridable). CO converts to Draft before expiry.
- `converted` state is **not terminal** — same pre-approval can generate multiple Drafts (e.g., if a prior Draft was withdrawn or rejected).
- Pre-approval is **pre-Origination** — does not write to `state_high_water_mark`.

**Entry points**:
- **BOS** (Branch Operations System): create new pre-approval (master data fetch from DaVinci → Campaign Eligibility Pre-Build → pre-approval screen); also access existing approved pre-approval for conversion.
- **CO Worklist**: access-only — surfaces approved pre-approvals for conversion to Draft. Cannot create a new pre-approval.

**Master data fetch** (added during session): Before the pre-approval screen opens, BOS fetches person detail, contract detail, and collateral data from DaVinci. This data is displayed as read-only context for the CO and passed to Campaign Eligibility Pre-Build as input. A snapshot is stored on the pre-approval record at creation.

---

### New: Features under Pre-Approval Capability

| File | Description |
|---|---|
| `capabilities/pre-approval/features/FEATURE_pre-approval-request-creation.md` | CO selects a restructure plan from pre-built results. Pre-condition: master data fetch (DaVinci) + Campaign Eligibility Pre-Build must both succeed before screen opens. Master data snapshot stored on pre-approval record. EasyPass auto-converts; non-EasyPass persists in `created`. |
| `capabilities/pre-approval/features/FEATURE_approval-request.md` | Non-EasyPass path only. Pre-approval submitted to designated approver. Outcomes: Approve or Reject. Approver may revise own decision before `converted`. CO may not influence decision. Notification via Sensei deferred (out of scope for 1.3). |
| `capabilities/pre-approval/features/FEATURE_pre-approval-expiry-management.md` | Non-EasyPass path only. System-configured default expiry duration. Approver may override at approval time. System auto-transitions `approved` → `expired`. Secondary expiry guard at conversion time. EasyPass carries no expiry. |
| `capabilities/pre-approval/features/FEATURE_draft-initializer.md` | Converts a valid pre-approval into a Draft application. Pre-populates form with selected plan, existing loan reference, and master data snapshot. Sets `easypass_flag`, `pre_approval_id`, `pre_approval_snapshot` on application record. Accessible via BOS or CO Worklist (non-EasyPass); EasyPass auto-triggers. Pre-approval remains reusable after conversion. |

---

### Modified: Underwriting Workflow Capability

**File**: `capabilities/underwriting-workflow/CAPABILITY.md`

| Addition | Detail |
|---|---|
| Topology D row in Shared Engine Topologies table | Pre-Approval — 6 states: `created`, `pending_approval`, `approved`, `rejected`, `expired`, `converted` |
| Topology E row in Shared Engine Topologies table | Restructure Application — combined view: Topology D (pre-approval) → Topology A (underwriting) with EasyPass routing and change detection |
| EasyPass Approval Routing at `pending_approval` | Configurable execution step: EasyPass + local authority → auto-approve; non-EasyPass → standard approver queue |
| Change Detection at Draft Submission | Configurable execution step (non-EasyPass restructure only): if `pre_approval_id` present + `easypass_flag = false` + no delta from `pre_approval_snapshot` → skip `pending_approval` |
| Topology D state diagram (Mermaid) | Separate EasyPass and Non-EasyPass state paths shown |
| Topology E entry flow diagram (Mermaid flowchart) | Flowchart showing BOS and CO Worklist entry points, creation path, approval path, and confirmation (approved) stage — distinct design from Topology D state diagram |
| Entry Points section for Topology A | 4 named entry paths: standard, restructure EasyPass via pre-approval, restructure non-EasyPass via pre-approval, restructure direct (no pre-approval) |
| Stage Flow section | Named end-to-end paths (A–D) for restructure applications from pre-approval through to funded |

---

### Modified: Loan Campaign Configuration Capability

**File**: `capabilities/loan-campaign-configuration/CAPABILITY.md`

Added `restructure` campaign type to the Campaign Types section. Restructure campaigns: target existing active borrowers, create a new Onigiri application using the campaign's Application Template, participate in Campaign Eligibility Pre-Build (eligibility criteria include DPD, loan status, prior restructure count, loan age, outstanding balance, and Risk Strategy). EasyPass flag is returned by pre-built — not stored on the campaign configuration.

---

### Modified: Smart Form Capability

**File**: `capabilities/smart-form/CAPABILITY.md`

Removed **Dynamic Field Options** feature. Pre-approval plan selection uses a dedicated screen — not Smart Form. The dedicated screen renders eligible campaigns directly from the Campaign Eligibility Pre-Build response with client-side EasyPass and tenor filters. Smart Form is not involved in pre-approval plan selection.

---

### Supporting Research Document

**File**: `artifacts/research/restructure-concept.md`

Master research document capturing all design decisions, scope boundaries, feature inventory, workflow paths, files created or modified, and open dependencies for the pre-approval capability.

---

## Decisions and Rationale

| Decision | Rationale |
|---|---|
| Pre-approval is a new **Capability** within Onigiri, not a new Product | Everything post-Draft reuses existing Onigiri workflow infrastructure. No distinct value proposition, lifecycle, or ownership boundary that would warrant a separate product. |
| Topology D on shared Underwriting Workflow engine | Avoids duplicate state machine infrastructure. Audit trail, execution step pattern, and transition atomicity are inherited at no additional cost. |
| EasyPass flag owned by pre-built, not campaign configuration | EasyPass reflects a dynamic authority assessment at evaluation time, not a static campaign property. Campaign configuration defines eligibility criteria; pre-built evaluates them against the customer context. |
| `converted` is not terminal | A pre-approval may need to generate multiple Drafts if prior attempts are withdrawn or rejected. Marking `converted` terminal would force unnecessary re-approval cycles. |
| CO Worklist as second entry point (read/convert only) | COs should not have to navigate back through BOS Customer Detail to action an already-approved pre-approval. CO Worklist surfaces actionable items directly. Creation remains BOS-only (requires customer context to call pre-built). |
| Master data fetch from DaVinci added to pre-approval entry | Pre-Build requires contract and collateral context to evaluate restructure eligibility criteria. Fetching this data upfront also populates the CO's read-only context panel — no redundant API calls. |

---

## Documents Updated This Session

- `capabilities/pre-approval/CAPABILITY.md` — created
- `capabilities/pre-approval/features/FEATURE_pre-approval-request-creation.md` — created
- `capabilities/pre-approval/features/FEATURE_approval-request.md` — created
- `capabilities/pre-approval/features/FEATURE_pre-approval-expiry-management.md` — created
- `capabilities/pre-approval/features/FEATURE_draft-initializer.md` — created
- `capabilities/underwriting-workflow/CAPABILITY.md` — modified (Topology D + E, entry points, stage flow, EasyPass routing, change detection)
- `capabilities/loan-campaign-configuration/CAPABILITY.md` — modified (added `restructure` campaign type)
- `capabilities/smart-form/CAPABILITY.md` — modified (removed Dynamic Field Options)
- `artifacts/research/restructure-concept.md` — created
