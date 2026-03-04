# Portfolio: Operations

**Strategic Domain**: Branch Operations, Document Processing, and Field Staff Productivity
**Business Owner**: COO / Head of Operations
**Status**: Active Investment
**Last Updated**: 2026-03-04

---

## Investment Thesis

The Operations portfolio groups the three systems that directly support field staff and document operations daily:

- **Matcha** — the universal document verification engine, shared across all portfolios as a cross-cutting service. Any system requiring document verification submits a task to Matcha.
- **Wasabi** — the AI pre-verification layer that runs before Matcha tasks are created, providing early warnings to underwriters and AI routing decisions to Matcha.
- **Sensei** — the centralized branch worklist and task orchestrator. Field staff access all their work — from collection calls to document collection tasks — in one place.

These products belong in the same portfolio because:
- They are all consumed primarily by branch operations staff (COs, supervisors, QA verifiers).
- They share a common dependency on **DaVinci** (Platform Portfolio) for customer identity and contact compliance.
- Their performance metrics combine into a single operational efficiency story: fewer human QA touches, higher CO throughput, and full contact compliance.
- Matcha and Wasabi are architecturally coupled (Wasabi fetches config from Matcha; Matcha uses Wasabi results for routing).

**Shared service boundary**: Matcha is the only product in this portfolio that serves *other* portfolios (Credit, and future Insurance/KYC portfolios). Its configuration API and task creation API are the integration surface. Governance over Matcha's config belongs to Operations, not to consuming portfolios.

---

## Constituent Products

| Product | Codename | Status | PRODUCT.md | Description |
|---------|----------|--------|------------|-------------|
| Document Verification Service | Matcha (抹茶) | ✅ Active | [PRODUCT](matcha/PRODUCT.md) | Universal document verification engine. Cross-portfolio shared service. |
| AI Document Verification | Wasabi (わさび) | 📝 Draft | [PRODUCT](wasabi/PRODUCT.md) | Stateless LLM-based verification engine. Matcha's AI routing layer. |
| Branch Operations Orchestration | Sensei (先生) | 📝 Draft | [PRODUCT](sensei/PRODUCT.md) | Centralized branch worklist, playbook engine, task tracker. |

---

## Portfolio-Level OKRs

| Objective | Key Result | Target |
|-----------|-----------|--------|
| Reduce QA headcount per verified document | % of Matcha tasks auto-completed by AI | > 70% by 12 months |
| Improve AI verification quality | False positive rate on AI auto-verified tasks (spot-check) | ≤ 0.01% |
| Increase CO throughput | Avg. tasks completed per CO per day | > 350 |
| Enforce contact compliance | % of tasks with zero daily contact limit violations | 100% |
| Reduce document errors reaching QA | % of Matcha tasks that route to human QA due to type mismatch (Wasabi catchable) | < 5% |

---

## Cross-Product Dependencies

```
Operations Portfolio → Platform Portfolio
  Matcha → DaVinci     : CustomerVerified, CustomerKYCExpired events
  Sensei → DaVinci     : ContactRecorded events (feedback loop)
  Sensei → DaVinci     : Subscribes to ContactLimitReached, ContactWindowClosed events
  Sensei → DaVinci     : customer.resolution_required → creates Admin tasks

Operations Portfolio ← Credit Portfolio
  Matcha ← Onigiri     : POST /task (document verification after facility creation)
  Wasabi ← Onigiri     : async verification during Draft phase

Operations Products ↔ within Portfolio
  Wasabi → Matcha      : Fetches DocumentVerificationItem config per documentTypeKey
  Matcha ← Wasabi      : Receives AI results attached to task creation payload

Operations Portfolio → External
  Matcha → Solomon     : work-entry / work-done events for QA distribution
  Matcha → Haibara     : Car check result callbacks
  Sensei → 3CX         : Call log verification queries
```

---

## Strategic Roadmap

| Horizon | Initiative | Owner |
|---------|-----------|-------|
| **Now** | Matcha — AI-First Verification capability (Wasabi integration, Verification Router) | Operations PO |
| **Now** | Wasabi — Core pipeline implementation (4 stages) | AI/ML Team |
| **Next** | Sensei — Task Engine + Work Queue (one-by-one processing mode) | Operations PO |
| **Next** | Sensei — Playbook Engine (HQ templates, branch variants, compliance locks) | Operations PO |
| **Later** | Sensei — Performance Dashboard (supervisor + self-service views) | Operations PO |
| **Later** | Matcha — Rule-based routing override (high-value, high-risk tasks) | Operations PO |
| **Later** | Wasabi — Model retraining pipeline (feedback loop from Matcha override data) | AI/ML Team |

---

## Key Risks and Constraints

| Risk | Severity | Mitigation |
|------|----------|-----------|
| Matcha config governance — consuming portfolios may want to change verification rules unilaterally | High | Matcha config API is Operations-owned. All config changes go through Operations PO. SLA for config changes: 2 business days. |
| Wasabi 99.99% confidence threshold limits auto-verification rate at launch | Medium | Expected auto-verification rate at launch: ~30%. Rate increases as model is fine-tuned via spot-check overrides. Accept lower rate initially; track via OKR. |
| Sensei contact compliance relies on DaVinci event latency | Medium | DaVinci emits events in near-real-time. If DaVinci event lag > 1 hour, Sensei may show stale contact counts. Monitor DaVinci event processing p95 latency. |
| Matcha is a cross-portfolio shared service — any downtime affects Credit (Onigiri) and future portfolios | High | Matcha SLA: 99.9% uptime. Async task creation (Onigiri Worker pattern) decouples Matcha availability from loan workflow transitions. |
