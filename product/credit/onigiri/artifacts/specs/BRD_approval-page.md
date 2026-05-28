# Business Requirement: Approval Page

**Product**: Onigiri — Loan Origination System
**Capability**: [Application Approval](../../capabilities/application-approval/CAPABILITY.md)
**Document Type**: Business Requirement Document (BRD)
**Author**: chanatan.sri@turbo.co.th
**Date**: 2026-05-25
**Status**: Draft

---

## 1. Objective

The Approval Page enables authorised credit staff to review and action loan applications that have completed risk assessment and are pending a human approval decision. The page serves two distinct but related functions:

**Primary — Application Approval Worklist**
Provide each approver with a queue of loan applications that have been assigned to them and are waiting for their decision. The worklist displays only applications distributed to the logged-in user by the **Worklist Distribution System** — an external system responsible for assigning applications to approvers within the correct `group_role`. Onigiri renders what the distribution system assigns; it does not own the assignment logic.

> **Dependency**: The worklist distribution logic is owned by an external system. The full distribution specification is pending and will be incorporated into this BRD when provided. The criteria below describe Onigiri's rendering and access-control responsibilities; distribution behaviour is out of scope until the dependency is resolved.

**Secondary — Pre-Approval Requests (Restructure)**
Provide CA-level approvers with a separate, accessible entry point to review restructuring pre-approval requests submitted by the sales team, without mixing them into the main application queue. Pre-approval requests are **not subject to the Worklist Distribution System** — they are routed directly to all users with `CA` group_role and managed entirely within Onigiri.

**Business Outcomes:**
- Eliminate manual queue management — each approver sees exactly what they are responsible for, automatically.
- Reduce approval cycle time — approvers have all the information needed to decide on a single screen without consulting other systems.
- Ensure accountability — every approval, rejection, or document request is permanently recorded with the actor's identity, role, and timestamp.
- Support role-appropriate confidentiality — sensitive financial and bureau data is visible only to roles with the authority to act on it.

---

## 2. Acceptance Criteria

### 2.1 Worklist — Role Scoping and Distribution

The worklist must show each approver only the applications they are authorised and assigned to action. Scoping is a two-layer control: the external Worklist Distribution System assigns applications to approvers, and Onigiri enforces a secondary role-boundary check to prevent cross-role exposure regardless of what the distribution system sends.

| # | Criterion |
|---|-----------|
| AC-01 | The worklist displays only applications that have been **assigned to the logged-in user by the Worklist Distribution System** and are currently in `pending_approval` state. |
| AC-02 | Onigiri enforces a secondary role-boundary check: even if the distribution system assigns an application to a user, Onigiri will not render it if the user's `group_role` does not match the application's `required_approver_role`. This prevents rendering errors caused by incorrect distribution assignments. |
| AC-03 | A user with `SALE_BRANCH` group_role cannot see applications requiring `SALE_AREA`, `CA`, or any other role — regardless of what the distribution system sends. |
| AC-04 | A user with no recognised approver `group_role` receives an access-denied response and cannot view the worklist. |
| AC-05 | Applications that have exited `pending_approval` (approved, rejected, returned to draft) are removed from the worklist regardless of distribution system state. |

> **Note — Distribution scope**: The criteria above describe Onigiri's rendering and access-control layer. The rules governing *which applications are assigned to which approver* (load balancing, geographic assignment, specialisation, manual override) are owned by the external Worklist Distribution System and are not specified here. This section will be updated once the distribution specification is provided.

> **Note — Pre-approval exclusion**: The pre-approval modal (AC-23 to AC-28) is not affected by the Worklist Distribution System. Pre-approval requests are routed to all `CA` group_role users directly within Onigiri.

### 2.2 Worklist — Display

The worklist must present enough information for an approver to prioritise and identify applications at a glance, without requiring them to open each one. Oldest applications are surfaced first to prevent cases from ageing unnoticed.

| # | Criterion |
|---|-----------|
| AC-06 | Each row displays: Application Reference, Borrower Full Name, Product / Campaign Name, Requested Loan Amount, Risk Level, and Days Pending (days since the application entered `pending_approval`). |
| AC-07 | The worklist is sorted by Days Pending descending (oldest applications at the top) by default. |
| AC-08 | The total count of pending applications assigned to the logged-in user is displayed in the page header. |
| AC-09 | When the worklist is empty (no applications distributed to the user), the page shows a clear empty-state message and does not show an error. |

### 2.3 Application Detail View — Access

The detail view is the approver's primary decision surface. It must be reachable directly from the worklist and must accurately reflect the application's current state at the moment it is opened. The view is read-only to preserve data integrity — all edits belong to the origination stage, not the approval stage.

| # | Criterion |
|---|-----------|
| AC-10 | An approver can open the full application detail by clicking any row in the worklist. |
| AC-11 | The detail view is read-only — no fields can be edited from this page. |
| AC-12 | If the application is no longer in `pending_approval` when the detail is opened (e.g., actioned by another approver), a banner is displayed showing the current status and the decision taken. The decision panel is disabled. |

### 2.4 Application Detail View — Role-Based Data Visibility

Sensitive financial, bureau, and risk data must only be visible to roles with the authority to act on that information. Each `group_role` has a defined visibility tier — higher tiers include more confidential data. Fields outside the approver's tier must not appear on the page at all, not merely be obscured, to eliminate any risk of inadvertent exposure.

| # | Criterion |
|---|-----------|
| AC-13 | All approver roles can see the application header and all Smart Form input fields (borrower, guarantor, loan setup, documents). |
| AC-14 | `SALE_AREA` approvers and above can additionally see declared financial fields: income, expenses, net income, total debt obligations, DTI, and LTV. |
| AC-15 | `CA` approvers and above can additionally see: NCB credit bureau result (full report), risk assessment output (risk level, deviation flags, evaluation trace), and collateral valuation from Dashi. |
| AC-16 | `CA+` approvers can additionally see the full workflow audit trail (all prior state transitions, decisions, and return-to-draft reasons). |
| AC-17 | Data fields belonging to a tier above the logged-in user's role are not present in the page at all — they are not hidden behind a blur or lock icon, they are simply absent. |

### 2.5 Approval Decision Actions

Every decision taken on an application must produce a deterministic, irreversible workflow outcome and a permanent audit record. The three available actions — Approve, Reject, and Request Document Upload — cover all valid paths forward from `pending_approval`. Concurrent decision attempts must be handled safely to prevent duplicate or conflicting state transitions.

| # | Criterion |
|---|-----------|
| AC-18 | Three decision actions are available on the detail page: **Approve**, **Reject**, and **Request Document Upload**. |
| AC-19 | **Approve**: On confirmation, the application transitions to `create_facility`. An approval snapshot (approver ID, role, approved amount, approved term, timestamp) is permanently recorded. |
| AC-20 | **Reject**: Requires the approver to enter a rejection reason (non-empty text) before confirming. On confirmation, the application transitions to `rejected`. |
| AC-21 | **Request Document Upload**: Requires the approver to specify at least one document type. On confirmation, the application returns to `draft` and the originating CO is notified. |
| AC-22 | If two approvers attempt to action the same application simultaneously, only the first submission succeeds. The second approver receives a notification that the application has already been decided, and the page refreshes to the current state. |
| AC-23 | All decisions are permanently recorded with: approver user ID, `group_role`, action type, timestamp, and reason/notes. No decision can be edited or deleted after submission. |

### 2.6 Pre-Approval Requests Modal (CA only)

Restructuring pre-approval requests are a separate workstream from loan applications and must not be mixed into the main worklist. CA approvers need a dedicated, easily accessible entry point on the same page to review and action these requests. The modal keeps both workstreams visible on one screen without navigating away.

> Pre-approval requests are not subject to the Worklist Distribution System. All criteria in this section are self-contained within Onigiri.

| # | Criterion |
|---|-----------|
| AC-24 | A **"Pre-Approval Requests"** button with a live pending-count badge is displayed on the worklist page for users with `CA` group_role only. Users with other roles do not see this button. |
| AC-25 | Clicking the button opens a modal overlay without navigating away from the worklist. |
| AC-26 | The modal list displays: Contract Number, Full Name, Branch Name, and Request Date/Time for each pending pre-approval request. |
| AC-27 | The modal includes a search bar that filters the list in real-time by Contract Number or Full Name. |
| AC-28 | Clicking a row in the modal navigates to the Pre-Approval Detail View for that request. |
| AC-29 | Pre-approval requests that have been actioned (no longer `PENDING_CA_REVIEW`) are removed from the modal list within 30 seconds. |

---

## 3. UAT Approach

### 3.1 Test Personas

| Persona | group_role | Purpose in UAT |
|---------|------------|----------------|
| **Somchai** | `SALE_BRANCH` (CO) | Validates low-risk application approval flow |
| **Narin** | `SALE_AREA` (AM) | Validates mid-risk approval and financial data visibility |
| **Prayut** | `CA` | Validates CA-level approval, bureau/RAE data visibility, and pre-approval modal |
| **Wiriya** | `CA+` (CRO) | Validates supervisory audit trail access |
| **Malee** | *(no approver role)* | Validates access denial |

---

### 3.2 Test Scenarios

#### Scenario 1 — SALE_BRANCH approves a low-risk application
**Persona**: Somchai (CO, `SALE_BRANCH`)
**Pre-condition**: An application exists in `pending_approval` with `required_approver_role = SALE_BRANCH` and risk level 1–10.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Somchai logs in and navigates to the worklist | Worklist shows only applications distributed to Somchai by the Worklist Distribution System. All visible applications have `required_approver_role = SALE_BRANCH`. No CA or AM-level applications are visible. |
| 2 | Somchai opens the application detail | Page renders. Sections visible: Application Header, Borrower Info, Guarantor Info, Loan Setup, Documents. Financial summary (Tier 3), NCB result (Tier 4), RAE trace (Tier 5), Collateral (Tier 6), Audit Trail (Tier 7) are **not present** on the page. |
| 3 | Somchai clicks **Approve** and confirms | Application transitions to `create_facility`. Approval snapshot recorded. Somchai is returned to the worklist with a success message. The approved application is no longer in the list. |

---

#### Scenario 2 — CA approves a CA-level application
**Persona**: Prayut (CA, `CA` group_role)
**Pre-condition**: An application exists in `pending_approval` with `required_approver_role = CA` and risk level 21–70.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Prayut logs in and navigates to the worklist | Worklist shows only applications requiring `CA`. |
| 2 | Prayut opens the application detail | All sections visible including NCB Credit Bureau Result (Tier 4), Risk Assessment Result with evaluation trace (Tier 5), and Collateral Valuation (Tier 6). Audit Trail (Tier 7) is **not present**. |
| 3 | Prayut reviews the RAE evaluation trace | The trace renders as a collapsible tree showing Strategy → Policy → Rule → Result for each evaluated policy. |
| 4 | Prayut clicks **Reject**, enters a reason, and confirms | Application transitions to `rejected`. Rejection reason recorded. Prayut returned to worklist. |

---

#### Scenario 3 — Approver requests document upload
**Persona**: Narin (AM, `SALE_AREA`)
**Pre-condition**: An application exists in `pending_approval` with `required_approver_role = SALE_AREA`.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Narin opens the application detail | Financial summary (Tier 3) is visible. NCB and RAE data (Tier 4–6) are **not present**. |
| 2 | Narin clicks **Request Document Upload** without selecting a document type | Submission is blocked. Validation message: at least one document type must be specified. |
| 3 | Narin selects a document type and confirms | Application returns to `draft`. The originating CO receives a notification. The application is removed from Narin's worklist. |

---

#### Scenario 4 — Role boundary enforcement
**Persona**: Somchai (CO, `SALE_BRANCH`) and Prayut (CA)

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Somchai logs in | Somchai's worklist shows only applications distributed to him. No CA-level applications are visible, even if the distribution system erroneously sends one. |
| 2 | Somchai attempts to open the detail URL of a CA-level application directly | System returns access denied — `group_role` mismatch. The page does not render. |
| 3 | Prayut logs in | Prayut's worklist shows only applications distributed to him. No `SALE_BRANCH`-level applications are visible. |

---

#### Scenario 5 — Concurrent approval attempt
**Persona**: Two CA users (Prayut and a colleague) with the same `CA` group_role

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Both Prayut and colleague open the same application detail simultaneously | Both see the decision panel as active. |
| 2 | Prayut clicks Approve and confirms first | Application transitions to `create_facility`. Prayut is returned to the worklist. |
| 3 | Colleague clicks Approve and confirms (after Prayut) | System detects the conflict. Colleague sees an error: "This application has already been decided." The page refreshes to show the approved state. No duplicate transition occurs. |

---

#### Scenario 6 — Pre-Approval Requests modal (CA only)
**Persona**: Prayut (CA); Somchai (CO, `SALE_BRANCH`)

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Somchai logs in and navigates to the worklist | The "Pre-Approval Requests" button is **not visible** to Somchai. |
| 2 | Prayut logs in and navigates to the worklist | The "Pre-Approval Requests" button is visible with a badge count of pending pre-approvals. |
| 3 | Prayut clicks the button | Modal opens without leaving the worklist. Modal shows Contract Number, Full Name, Branch Name, Request Date/Time for each pending item. |
| 4 | Prayut types a contract number in the search bar | Modal list filters in real-time to matching contracts. |
| 5 | Prayut clicks a row | Modal closes. Prayut is navigated to the Pre-Approval Detail View for that request. |

---

#### Scenario 7 — No approver role (access denial)
**Persona**: Malee (no approver group_role)

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Malee attempts to navigate to the worklist | System returns access denied. The worklist page does not render. |

---

### 3.3 UAT Sign-Off Criteria

The Approval Page is considered UAT-passed when all of the following are met:

- All 7 test scenarios completed with expected results across all personas.
- No Severity-1 or Severity-2 defects remain open.
- Role-based data visibility confirmed for all 4 group_roles (SALE_BRANCH, SALE_AREA, CA, CA+).
- Concurrent approval conflict handling confirmed (Scenario 5).
- Pre-approval modal confirmed visible to CA only and non-visible to other roles (Scenario 6).
- All approval decisions confirmed as immutably recorded in the audit log by the testing team.
