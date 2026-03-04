# Capability: Template Library

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Provide a library of reusable action type definitions that standardize how work is described, measured, and escalated across the organization — owned by HQ, referenced by Playbook Steps, and consumed by the Task Engine for outcome validation and SLA enforcement.

## Why It Exists (First Principles)

- **Standardization**: Without a shared library, each playbook could define "Call" differently (different outcomes, different required fields, different SLAs). The Template Library enforces consistency.
- **Reusability**: Action types and their outcome definitions are the same whether used in a delinquency recovery playbook or an insurance renewal playbook. Define once, reference everywhere.
- **Governance**: HQ owns the definitions. Supervisors cannot modify templates directly — they customize through Playbook Variants, which reference (not copy) template definitions.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Action Type Registry | Draft | HQ manages list of available action types (Call, Visit, Admin, etc.) with their properties |
| Outcome Definition | Draft | Each action type has a defined set of typed outcomes |
| Required Fields per Outcome | Draft | Each outcome specifies which fields must be filled on task completion (e.g., PTP requires amount + date) |
| SLA Defaults | Draft | Each action type has a default SLA duration used when a task or playbook step doesn't specify one |
| Escalation Rules | Draft | Auto-escalation rules per action type (e.g., after 3 failed Call outcomes → escalate to Visit) |

---

## Business Rules

### Action Type Definitions

| Action Type | Typed Outcomes | Required Fields on Specific Outcomes |
|-------------|----------------|--------------------------------------|
| 📞 Call | PTP, No Answer, Refused, Callback, Wrong Number, Line Busy, Voicemail | PTP → PTP amount, PTP date; Callback → scheduled date/time |
| 🏠 Visit | Met Customer, Not Home, Address Invalid, PTP (in-person), Refused | PTP → PTP amount, PTP date; Address Invalid → new address |
| 📋 Admin | Completed, Incomplete, Escalated | Escalated → escalation reason |
| ⏳ Wait | (auto-advances; no manual outcome) | — |
| 🔔 Notify Supervisor | Acknowledged, No Response | — |
| 📧 Send Notification | (system-dispatched; Delivered/Failed) | — |

### SLA Defaults by Action Type

| Action Type | Default SLA |
|-------------|-------------|
| 📞 Call | 4 hours |
| 🏠 Visit | 8 hours |
| 📋 Admin | 24 hours |
| ⏳ Wait | Duration defined in step configuration |

### Escalation Rule Example

After 3 failed Call outcomes (e.g., 3× No Answer or Refused) → auto-escalate to Visit step in playbook.
Escalation rules are defined in the Template Library per action type and referenced by Playbook Steps.

### Governance Rule

- Templates are owned and managed by HQ
- Supervisors reference templates via Playbook Steps — they cannot modify template definitions
- If a supervisor needs a custom outcome or field, they must request HQ to update the Template Library
- Changes to templates take effect in new playbook instances; in-flight instances continue with the previous definition

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| HQ-only modification | Template Library modifications restricted to HQ admin role |
| Reference stability | Existing playbook steps referencing a template must not break when template is updated (versioning required) |
| Required field enforcement | Task Engine must validate required fields on task closure based on template definition |
