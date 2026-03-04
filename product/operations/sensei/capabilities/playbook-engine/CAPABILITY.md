# Capability: Playbook Engine

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Provide a structured, reusable system for defining multi-step operational strategies (Playbooks) that drive how field staff handle specific business objectives (delinquency recovery, insurance renewal, KYC re-verification). HQ defines System Templates; branch supervisors fork Branch Variants with controlled customization within compliance guardrails.

## Why It Exists (First Principles)

- **Policy Alignment**: Thousands of branch staff across hundreds of branches must execute consistent strategies. Without structured playbooks, each branch invents its own approach, causing inconsistent outcomes and compliance risk.
- **Knowledge Codification**: Effective collection strategies and renewal workflows are institutional knowledge. Playbooks capture this as executable templates, not tribal knowledge.
- **Adaptability**: HQ defines the default strategy, but local conditions require branch-level customization within guardrails.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Playbook Builder (HQ) | Draft | HQ creates System Templates with step sequences, action types, timing, outcome transitions, and compliance locks |
| Branch Variant Fork | Draft | Supervisors fork a System Template into a Branch Variant with drag-and-drop step editing within allowed rules |
| Compliance Lock Enforcement | Draft | Locked steps (🔒) cannot be removed, reordered past their boundary, or have outcome transitions modified |
| Outcome Transition Router | Draft | Each step defines per-outcome transitions (Next Step / Specific Step / Retry / End / Escalate) |
| Template Version Sync | Draft | HQ publishes new template version; branches with variants notified to review and merge changes |
| Playbook Instantiation | Draft | When a playbook is triggered for a customer, step tasks are created in Task Engine |

---

## Business Rules

### Step Action Types

| Action Type | Icon | Possible Outcomes |
|-------------|------|------------------|
| Call | 📞 | PTP, No Answer, Refused, Callback, Wrong Number, Line Busy, Voicemail |
| Visit | 🏠 | Met Customer, Not Home, Address Invalid, PTP (in-person), Refused |
| Admin | 📋 | Completed, Incomplete, Escalated |
| Wait | ⏳ | Auto-advances after duration (no manual outcome) |
| Notify Supervisor | 🔔 | Acknowledged, No Response |
| Send Notification | 📧 | (auto-dispatched; delivered/failed) |

### Outcome Transition Types

| Transition | Meaning |
|-----------|---------|
| `→ Next Step` | Continue to next step in sequence (default) |
| `→ Specific Step` | Jump to a named step |
| `→ Retry` | Repeat same step with max attempt limit |
| `→ End (Success)` | Close playbook as succeeded |
| `→ End (Failed)` | Close playbook as failed |
| `→ Escalate` | Route to supervisor for manual decision |

### Playbook Hierarchy

| Level | Owner | Can Edit? |
|-------|-------|-----------|
| System Template | HQ | HQ only; read-only for branches |
| Branch Variant | Branch Supervisor | Yes, within allowed edit rules |

### Supervisor Edit Rules

| Allowed | Not Allowed |
|---------|-------------|
| Reorder steps (drag and drop) | Delete 🔒 locked steps |
| Drag outcome transitions to different target steps | Edit System Templates directly |
| Add optional steps | Remove audit trail / compliance logging |
| Add/remove outcomes on non-locked steps | Modify outcome transitions on 🔒 locked steps |
| Adjust wait timing (within limits) | Reorder locked steps past their compliance boundary |
| Change assignee rules | Bypass publishing workflow |
| Set retry limits on outcomes | |
| Remove non-locked steps | |

### Compliance-Locked Steps

Locked steps (🔒) are steps that HQ marks as required for legal or operational compliance:
- Cannot be removed from the playbook
- Cannot be reordered past a defined boundary (e.g., "ส่งรายงาน" must always be last)
- Timing adjustable within limits set by HQ
- Outcome transitions visible but not modifiable by supervisors

---

## Example Playbook: ติดตามหนี้ค้างชำระ

| Step | Action | Timing | Key Outcome Transitions | Lock |
|------|--------|--------|------------------------|------|
| 1 | 📞 โทรศัพท์ครั้งที่ 1 | Day 1 | PTP → End ✅; No Answer → Retry (max 3) then → Step 5; Refused → Step 6 | — |
| 2 | ⏳ รอ | 3 days | Auto → Next | — |
| 3 | 📞 โทรศัพท์ครั้งที่ 2 | Day 4 | PTP → End ✅; No Answer → Step 5 | — |
| 4 | ⏳ รอ | 3 days | Auto → Next | — |
| 5 | 🏠 ลงพื้นที่ | Day 7 | Met & PTP → End ✅; Not Home → Step 6 | — |
| 6 | 📋 ส่งรายงาน | Day 8 | Completed → Next | 🔒 |
| 7 | 🔔 แจ้งหัวหน้า | Auto | Acknowledged → End | 🔒 |

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Compliance lock integrity | Locked steps cannot be removed or reordered by any user except HQ |
| Version tracking | Branch variants must track which system template version they were forked from |
| Instantiation atomicity | Playbook instantiation (creating all tasks) must be atomic — all tasks created or none |
