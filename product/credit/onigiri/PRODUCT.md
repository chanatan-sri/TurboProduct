# Product: Loan Origination System

**Codename**: Onigiri (おにぎり)
**Portfolio**: Credit → [PORTFOLIO](../../PORTFOLIO.md)
**Status**: 📝 Draft
**Executive Owner**: CPO
**Last Updated**: 2026-03-04

> *Onigiri (おにぎり) — A tightly packed, self-contained unit. Like the rice ball, Onigiri wraps the entire loan origination lifecycle into a single, cohesive product — from application intake through underwriting to disbursement. Everything the borrower needs, held together in one place.*

---

## Problem Statement

Loan origination across multiple product types (car title, land title, personal loans) requires different eligibility rules, form fields, risk strategies, document requirements, and workflow execution steps per campaign. A rigid, hardcoded system cannot keep pace with regular product launches, seasonal promotions, and regulatory changes. Risk policies must sometimes change weekly. Every code deployment adds latency and operational risk.

---

## Value Proposition

A single configurable platform that governs the full loan application lifecycle — from smart form intake through a fixed-topology underwriting state machine to core banking disbursement — without code changes for new campaigns or risk rule modifications.

**For whom**: Branch Credit Officers (COs) and underwriters who originate loans daily; Product Managers who launch campaigns; Risk Officers who manage assessment strategies.

---

## Product Boundary

**This product IS responsible for:**
- Loan application intake (Smart Form — Page → Section → Field composability)
- Underwriting workflow state machine (Draft → Risk Assessment → Approval → Create Facility → ... → Funded)
- Loan campaign configuration (pricing, eligibility, form template, risk strategy, execution steps)
- Risk assessment engine (JMESPath-based Strategy → Policy → Rule hierarchy)
- Integration gateway to Matcha, Wasabi, DaVinci, Sensei, Core Banking, NCB at defined boundaries

**This product IS NOT responsible for:**
- Document verification logic or QA workflow (owned by **Matcha**)
- AI document analysis and report assembly (owned by **Wasabi**)
- Customer master data and Golden Record (owned by **DaVinci**)
- Branch task tracking and worklist management (owned by **Sensei**)
- Loan repayment scheduling, statements, or early settlement (future: **Loan Servicing** product)
- Delinquency management or collection workflows (future: **Collections** product)

**This product RECEIVES from:**
- DaVinci → customer identity + product summary on application creation → via REST API
- Wasabi → early-warning document verification report during Draft phase → via async callback
- Matcha → verification outcome (APPROVED/RETURNED/REFERRED) → via webhook callback
- NCB → credit bureau inquiry result → via API (triggered by OTP consent in Smart Form)
- Core Banking → fund transfer result (success / failure) → via webhook callback

**This product SENDS to:**
- Matcha → document verification task (POST /task) after Create Facility state → via REST API
- Wasabi → document image URL + expected type + system data on upload → via async API
- Sensei → TaskCreationRequest event when branch action is needed → via event
- Core Banking → Create Facility command, Create Loan + Disbursement command → via API
- NCB → credit bureau inquiry request → via API
- DaVinci → ApplicationCreated, ApplicationApproved, CustomerProfileUpdated events → via event

---

## Capability Registry

| Capability | Owner | Status | Description |
|-----------|-------|--------|-------------|
| [Smart Form](capabilities/smart-form/CAPABILITY.md) | Engineering | Draft | Configurable Page → Section → Field form. JSON data in DocumentDB. Savable mid-session. Covers Borrower, Guarantor, Loan Setup, Summary, Document Upload stages. |
| [Underwriting Workflow](capabilities/underwriting-workflow/CAPABILITY.md) | Engineering | Draft | Fixed-topology 4-phase state machine (Origination → Underwriting → Decision → Terminal). 11 states. Configurable execution steps inside each state. Cash vs. non-cash path divergence. |
| [Loan Campaign Configuration](capabilities/loan-campaign-configuration/CAPABILITY.md) | Product | Draft | Single configuration umbrella per loan product: pricing, eligibility rules, application template, risk strategy, workflow execution steps. Zero code changes for new campaigns. |
| [Risk Assessment Engine](capabilities/risk-assessment-engine/CAPABILITY.md) | Engineering | Draft | JMESPath-based configurable rule engine. Strategy → Policy → Rule hierarchy. Produces max risk level, deviation flags, conditional document requirements. Full evaluation trace for audit. |
| [Disbursement Orchestration](capabilities/disbursement-orchestration/CAPABILITY.md) | Engineering | Concept | Receives Matcha `approved` callback and Core Banking fund transfer callback to advance the application through `waiting_fund_transfer` → `waiting_create_loan_operation`. Owns post-document-verification disbursement states. |

---

## Product-Level User Flow

```mermaid
stateDiagram-v2
    direction LR

    state "Phase: Origination" as orig {
        Draft
    }

    state "Phase: Underwriting" as uw {
        RiskAssessment: Risk Assessment
        Approval: Approval + Risk Level
    }

    state "Phase: Decision" as dec {
        CreateFacility: Create Facility
        DocCheck: Pending Document Checking
        WaitFund: waiting_fund_transfer
        WaitLoan: waiting_create_loan_operation
        Cash: Cash?\n[TBD — post waiting_create_loan_operation]
        Confirmation1: Confirmation
        CreateLoanDisb1: Create Loan + Disbursement
        QA_top: QA
        QA_bottom: QA
        ConfirmationBottom: Confirmation
        CreateLoanDisbBottom: Create Loan + Disbursement
    }

    state "Phase: Terminal" as term {
        Funded
        Rejected
        Withdrawn
        Expired
        ReturnedForRevision: returned_for_revision
    }

    [*] --> Draft: Create application
    Draft --> RiskAssessment: Submit

    RiskAssessment --> Approval
    RiskAssessment --> Draft: Request additional documents

    Approval --> CreateFacility: Approve
    Approval --> Rejected: Reject
    Approval --> Draft: Request document upload

    CreateFacility --> DocCheck: Matcha task created

    DocCheck --> WaitFund: Matcha approved
    DocCheck --> ReturnedForRevision: Matcha returned
    DocCheck --> Approval: Matcha referred

    WaitFund --> WaitLoan: CB fund transfer success
    WaitFund --> ReturnedForRevision: Matcha re-decision returned
    WaitFund --> Approval: Matcha re-decision referred

    WaitLoan --> Cash: next state TBD

    Cash --> Confirmation1: y (cash)
    Confirmation1 --> CreateLoanDisb1: Create loan
    CreateLoanDisb1 --> QA_top
    QA_top --> Funded

    Cash --> QA_bottom: n (non-cash)
    QA_bottom --> ConfirmationBottom
    ConfirmationBottom --> CreateLoanDisbBottom: Create loan
    CreateLoanDisbBottom --> Funded

    QA_top --> Draft: Request document upload
    QA_bottom --> Draft: Request document upload

    Draft --> Withdrawn: Customer not interested
    Draft --> Expired: Application expired
```

---

## Integration Map

```mermaid
graph LR
    Onigiri[Onigiri\nLoan Origination]
    DaVinci[DaVinci\nMaster Data]
    Wasabi[Wasabi\nAI Verification]
    Matcha[Matcha\nDoc Verification]
    Sensei[Sensei\nBranch Worklist]
    CoreBanking[Core Banking]
    NCB[NCB\nCredit Bureau]

    DaVinci -->|Customer identity| Onigiri
    Onigiri -->|App events| DaVinci
    Onigiri -->|Document + type| Wasabi
    Wasabi -->|Verification report| Onigiri
    Onigiri -->|POST /task + Wasabi results| Matcha
    Matcha -->|Outcome webhook| Onigiri
    Onigiri -->|TaskCreationRequest| Sensei
    Sensei -->|TaskCompleted| Onigiri
    Onigiri -->|Create Facility| CoreBanking
    CoreBanking -->|Fund transfer result| Onigiri
    Onigiri -->|NCB inquiry| NCB
```

---

## Product-Level Metrics and KPIs

| Metric | Description | Target |
|--------|-------------|--------|
| Origination Cycle Time | Time from Draft creation to Funded state (p50) | < 5 business days |
| Underwriting Automation Rate | % of applications with no manual risk rule intervention post-approval | > 90% |
| Campaign Launch Time | Calendar days from product request to first live application (no-code) | < 2 business days |
| Document Error Rate at Intake | % of Matcha tasks that route to human QA due to type mismatch (Wasabi-catchable) | < 5% |
| Return Rate | % of applications returned from Approval or QA back to Draft | < 15% |

---

## Detailed Reference

For full capability specifications, business rules, and design decisions, see: [ATLAS.md](ATLAS.md)
