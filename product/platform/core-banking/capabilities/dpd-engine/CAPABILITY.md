# Capability: DPD Engine

**Product**: Core Banking — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Platform
**Product Owner**: TBD (Platform PO / Core Banking PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Maintain the authoritative Days Past Due (DPD) counter for every loan account, tick it daily for accounts with missed payments, reset it on full catch-up, and emit events when the DPD bucket changes. DPD is the single most important delinquency signal in the lending lifecycle — it drives collection assignment, NPL classification, provisioning, and regulatory reporting. Every system that needs to know "how overdue is this account?" relies on DPD Engine as the source of truth.

## Why It Exists (First Principles)

- **Authoritative delinquency signal**: Collections, Loan Servicing, DaVinci, and Risk Analytics all need a consistent DPD number. If each system computed DPD independently, inconsistencies would emerge (different holiday calendars, different payment timing interpretations). One engine eliminates this.
- **Downstream orchestration trigger**: DPD bucket changes are the primary trigger for collection assignment changes, provisioning adjustments, and regulatory NPL classification. These downstream reactions require a reliable, timely event.
- **Regulatory reporting**: BOT (Bank of Thailand) NPL classification uses DPD thresholds. An auditable DPD calculation is required for examination.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| DPD Counter | Draft | Per-account DPD integer counter. Increments by 1 each calendar day when a scheduled payment is missed. Authoritative — no other system computes DPD independently. |
| Daily DPD Tick | Draft | Scheduled daily job. For each active account: checks whether today's scheduled payment has been received. If not, increments DPD. Idempotent — safe to re-run. |
| DPD Reset | Draft | When a full catch-up payment clears all overdue instalments, DPD resets to 0. Partial payments do not reset DPD. |
| DPD Bucket Classifier | Draft | Maps DPD integer to a bucket (0, 1–30, 31–90, 91–180, 181+). Bucket change triggers event emission. |
| LoanDPDChanged Event | Draft | Emitted whenever the DPD bucket changes (0→1, 30→31, 90→91, etc.). Payload: account ID, previous bucket, new bucket, current DPD integer, effective date. Consumed by DaVinci → Collections → Sensei. |
| DPD Audit Log | Draft | Immutable log of every DPD change: account ID, date, previous DPD, new DPD, triggering event (missed payment, partial payment, full catch-up). |

---

## Business Rules

### DPD Calculation Rules

| Scenario | Rule |
|----------|------|
| Payment received on or before due date | DPD = 0. No increment. |
| Payment not received by end of due date | DPD increments by 1. |
| Partial payment received (does not clear all overdue) | DPD continues to increment. |
| Full catch-up payment received (clears all overdue instalments) | DPD resets to 0. |
| Account in grace period (product-configured) | DPD does not tick during grace period. |
| Public holiday | DPD does not tick if due date falls on a public holiday and payment is received by the next business day (product-configurable). |
| Account Settled | DPD freezes at settlement date value. No further ticks. |
| Account Written Off | DPD freezes at write-off date value. Emits final `LoanDPDChanged` event. |

### DPD Bucket Reference

| Bucket | DPD Range | Label | Downstream Use |
|--------|-----------|-------|----------------|
| 0 | 0 | Current | No collection action required |
| 1 | 1–30 | Early Overdue | Branch staff collection (per Collections assignment rules) |
| 2 | 31–90 | Mid Overdue | Field Collector assignment |
| 3 | 91–180 | Default | Specialist / legal escalation candidate |
| 4 | 181+ | Severe Default | Write-off review |

> **Note**: Bucket boundaries are the default. Collections' Assignment & Escalation Engine applies its own multi-factor rules on top of these buckets. DPD Engine only classifies — Collections decides what to do.

### NPL Classification Threshold

| Threshold | Classification | BOT Reporting |
|-----------|---------------|---------------|
| DPD ≥ 90 | Non-Performing Loan (NPL) | Must be reported to BOT in NPL schedule |
| DPD < 90 | Performing / Sub-standard | Standard provisioning applies |

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| DPD tick latency | Daily DPD tick job must complete for all active accounts within the processing window (configurable, default: by 06:00 local time). |
| Event delivery guarantee | `LoanDPDChanged` events must be delivered at-least-once with idempotent consumer support. |
| Single authority | No other system may maintain its own DPD counter. All DPD queries go to Core Banking. |
| Audit trail completeness | Every DPD change must have a corresponding audit log entry traceable to the triggering event. |

---

## Open Questions

- Is the grace period configured per product type in Core Banking, or per campaign in Onigiri's Loan Campaign Configuration? Ownership of this parameter must be decided.
- What is the exact cut-off time for "end of due date" — end of business day, midnight, or 23:59:59? This affects DPD for next-day payments.
- Should DPD bucket 0 (current) changes also emit a `LoanDPDChanged` event (e.g., when DPD resets from 5 back to 0), or only on bucket boundary crossings?
