# CHANGELOG_017 — Risk Level Routing Table Update

**Date**: 2026-05-25
**Author**: chanatan.sri@turbo.co.th
**Layer Affected**: Capability (Risk Assessment Engine, Application Approval) + Feature (Approval Routing Assignment)
**Branch**: loan-origination-with-cash

---

## What Changed

### Risk Level → Approval Authority: Range-Based Model

The risk level routing table has been replaced. The previous model used single numeric values (10, 20, 30…) as exact match keys. The new model uses **inclusive ranges** (`From RL` / `To RL`) and introduces:

- `group_role` system codes as the actual routing key written to `required_approver_role`
- Multiple approval positions per group (any one position may action — one approval required)
- Explicit auto-rejection threshold: **`risk_level > 70` → system auto-rejects**
- Three new positions: **DAM** (Deputy Area Manager), **SAM** (Senior Area Manager) added to `SALE_AREA`; **GOD** (system-level) for unconditional auto-reject at 99
- **CA+** group (`CRO`, 71–80): designated CRO authority tier, but auto-rejected in this phase — no human worklist entry

### Previous Table (replaced)

| Risk Level | Approver |
|------------|----------|
| 10 | CO / SCO / BM / SBM |
| 20 | AM |
| 30 | CA |
| 40 | CA Manager |
| 50 | CRO |
| 60 | CEO |
| 70 | Auto-decline |
| 99 | Auto-decline |

### New Table

| From RL | To RL | group_role | Approval Positions | Outcome |
|---------|-------|------------|-------------------|---------|
| 1 | 10 | `SALE_BRANCH` | CO, SCO, BM, SBM | Routes to approver worklist |
| 11 | 20 | `SALE_AREA` | DAM, AM, SAM | Routes to approver worklist |
| 21 | 70 | `CA` | CA | Routes to approver worklist |
| 71 | 80 | `CA+` | CRO | System auto-rejects |
| 99 | 99 | *(system)* | GOD | System auto-rejects |
| — | 0 | *(default)* | — | System auto-rejects |

---

## Auto-Rejection Rule (new)

**Risk level > 70 → system auto-rejects without human review.** Applications that score 71–80 (CA+ / CRO tier) or 99 (policy violation / GOD) bypass `pending_approval` and transition directly to `rejected` with a system-generated reason. This applies to the current product phase.

The previous documents implied auto-decline only at 70 and 99 as exact values. The new rule is a threshold — any value strictly above 70 triggers auto-rejection.

---

## Visibility Tier Alignment

The Application Approval visibility tier thresholds have been updated to align with the new group_role model:

| Tier | Old Minimum | New Minimum |
|------|------------|------------|
| Tier 3 (Financials) | AM and above (risk ≥ 20) | `SALE_AREA` and above (risk ≥ 11) |
| Tier 4–6 (Bureau, Risk, Collateral) | CA and above (risk ≥ 30) | `CA` and above (risk ≥ 21) |
| Tier 7 (Audit Trail) | CA Manager and above (risk ≥ 40) | `CA+` and above (risk ≥ 71) — supervisory view, not approval |

---

## Documents Modified

| File | Change Type |
|------|------------|
| `capabilities/risk-assessment-engine/CAPABILITY.md` | Updated — Risk Level to Approval Authority table replaced with range-based model |
| `capabilities/application-approval/CAPABILITY.md` | Updated — routing table and visibility tier thresholds aligned to new group_role model |
| `capabilities/application-approval/features/FEATURE_approval-routing-assignment.md` | Updated — mapping table replaced; auto-rejection threshold updated from "risk 70 or 99" to "risk > 70" |

---

## Open Questions Surfaced

- What is the exact behavior for risk levels **81–98**? The table defines 71–80 (CA+) and 99 (GOD) but has a gap at 81–98. Pending confirmation: are 81–98 also auto-rejected, or is this a data entry gap in the source table?
- Is **DAM** Deputy Area Manager or a different title? Confirm official position name.
- Is **SAM** Senior Area Manager? Confirm official position name.
- For the `CA+` / CRO tier (71–80): is this auto-rejection permanent (will always be auto-rejected), or is it a phase-specific rule that will later be changed to route to a human CRO approver?
