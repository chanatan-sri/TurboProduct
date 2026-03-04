# Portfolio: Platform

**Strategic Domain**: Shared Data Infrastructure and Enterprise Master Data
**Business Owner**: CTO / Head of Data Platform
**Status**: Active Investment
**Last Updated**: 2026-03-04

---

## Investment Thesis

The Platform portfolio holds shared infrastructure that all other portfolios depend on. It contains two foundational systems that together represent the complete enterprise data layer:

- **DaVinci** is the enterprise Golden Record — the single source of truth for customer identity, consent, and cross-product contact compliance.
- **Core Banking** is the authoritative financial ledger — the single source of truth for loan account state, balance, payment allocation, interest accrual, and DPD status.

Neither product has a direct customer-facing value proposition. Their value is measured by the cost, risk, and inconsistency they eliminate for other products:
- Without DaVinci: each product maintains its own customer record → PDPA consent cannot be enforced → Thai Debt Collection Act contact limits cannot be tracked across products.
- Without Core Banking: each product implements its own balance calculation and DPD logic → inconsistent NPL figures → regulatory exposure → Collections and Loan Servicing operate on stale or contradictory data.
- With both: one canonical customer identity, one authoritative financial ledger. Every downstream consumer queries Platform instead of duplicating this logic.

**Why a separate portfolio?** Platform products have fundamentally different investment and governance patterns: they are owned by engineering/data teams (not product teams), they have no direct revenue attribution, and their value is measured as shared infrastructure cost avoidance rather than feature delivery velocity. Conflating them with operational or domain portfolios would obscure this distinction.

**Future Platform candidates**: A shared notification service (SMS/push/email), a shared reporting data warehouse, or a workflow engine (if Sensei's task engine becomes generic enough) may be added to this portfolio as the company scales.

---

## Constituent Products

| Product | Codename | Status | PRODUCT.md | Description |
|---------|----------|--------|------------|-------------|
| Customer & Product Master Data | DaVinci (ダヴィンチ) | 📝 Draft | [PRODUCT](davinci/PRODUCT.md) | Enterprise Golden Record. Consent-based visibility, event-driven sync, data consolidation engine, resolution workflow. |
| Core Banking | TBD | 📝 Draft | [PRODUCT](core-banking/PRODUCT.md) | Authoritative financial ledger. Loan account lifecycle, payment hierarchy, interest & fee calculation, DPD engine. Single source of truth for all loan account state. |

---

## Portfolio-Level OKRs

| Objective | Key Result | Target |
|-----------|-----------|--------|
| Achieve complete Golden Record coverage | % of active customers with a DaVinci record | 100% by launch |
| Enable real-time data currency | DaVinci event processing latency (p95) | < 30 seconds |
| Enforce PDPA compliance | % of cross-entity data access attempts blocked without valid consent | 100% |
| Resolve data conflicts within SLA | T1 resolutions completed on same day | > 95% |
| Resolve data conflicts within SLA | T3 resolutions completed within 5 business days | > 90% |
| Eliminate contact frequency violations | % of collection contacts that exceed daily limit (per Debt Collection Act) | 0% |

---

## Cross-Product Dependencies

```
Platform Portfolio → All Portfolios

  Core Banking (owned)
  Core Banking ← Onigiri       : CreateFacility command, DisburseLoan command → via API
  Core Banking ← Payment GW    : PaymentReceived event → via event
  Core Banking ← Scheduler     : DailyAccrualTrigger → via scheduled job
  Core Banking → DaVinci        : LoanDisbursed, LoanPaymentReceived, LoanStatusChanged, LoanDPDChanged, LoanClosed events
  Core Banking → Onigiri        : CreateFacilityResult → via API response

  DaVinci (owned)
  DaVinci ← Core Banking  : LoanDisbursed, LoanPaymentReceived, LoanStatusChanged, LoanClosed, LoanDPDChanged events
  DaVinci ← Policy Admin  : PolicyIssued, PremiumPaid, PolicyLapsed, PolicyCancelled, PolicyRenewed events
  DaVinci ← Matcha        : CustomerVerified, CustomerKYCExpired events
  DaVinci ← Onigiri       : ApplicationCreated, ApplicationApproved, CustomerProfileUpdated events
  DaVinci ← Sensei        : ContactRecorded events

  DaVinci → Sensei         : ContactLimitReached, ContactLimitApproaching, ContactWindowClosed events
  DaVinci → Sensei         : customer.resolution_required events (triggers Admin tasks)
  DaVinci → Matcha         : T3 verification task requests (identity evidence)
  DaVinci → Branch Dashboards / Collection Systems / Risk Analytics : REST Query API
  DaVinci → Onigiri        : Customer identity lookup on application creation
  DaVinci → Collections    : LoanDPDChanged, ContactRecorded events (Collections as downstream consumer)
```

---

## Strategic Roadmap

| Horizon | Initiative | Owner |
|---------|-----------|-------|
| **Now** | DaVinci — Data Consolidation Engine (field authority, no-data-loss, admin override, self-check) | Platform PO |
| **Now** | DaVinci — Event-Driven Synchronization (Core Banking + Onigiri events) | Platform PO |
| **Now** | DaVinci — Golden Record + Downstream Query API | Platform PO |
| **Now** | Core Banking — PRODUCT.md defined; ARCHITECTURE.md and capability specs to follow | Platform PO |
| **Next** | DaVinci — PDPA Consent Engine (Consent Registry, visibility filtering API) | Platform PO |
| **Next** | DaVinci — Collection Contact Compliance (Unified Contact Log, Frequency Check API) | Platform PO |
| **Next** | DaVinci — Data Resolution Workflow (ResolutionRequest lifecycle, T1/T2/T3, Sensei integration) | Platform PO |
| **Next** | Core Banking — ARCHITECTURE.md: API contracts, data model, event schema registry | Core Banking PO |
| **Next** | Core Banking — Assign codename | Platform PO |
| **Later** | DaVinci — Collateral Master Data capability (gap identified in architecture gap analysis) | Platform PO |
| **Later** | DaVinci — Policy Admin event domain (insurance product sync) | Platform PO |
| **Later** | Core Banking — Feature decomposition under each capability | Core Banking PO |

---

## Key Risks and Constraints

| Risk | Severity | Mitigation |
|------|----------|-----------|
| DaVinci is a blocking dependency for all other portfolios | Critical | Prioritize Golden Record + Query API as the first deliverable. All other capabilities are additive. |
| Core Banking is a blocking dependency for Collections and Loan Servicing | Critical | Core Banking ARCHITECTURE.md and event schema must be defined before Collections or Loan Servicing integration work begins. |
| Architecture gap: PDPA Consent Engine not yet in ATLAS | High | Identified in `davinci/artifacts/research/atlas_vs_architecture_gap_analysis.md`. Must be specced before launch. |
| Architecture gap: Contact Compliance API contract not specified in ARCHITECTURE.md | High | Sensei depends on this for event subscription. Must be resolved before Sensei Task Engine is built. |
| Core Banking codename not assigned | Medium | A codename is needed before cross-team communication becomes confusing. Assign at next Portfolio review. |
| Architecture gap: Collateral Master Data not in ATLAS | Medium | Onigiri captures collateral data; it needs a DaVinci home for cross-product visibility. Defer to Later horizon. |
| `davinci_customer_id` vs. National ID as canonical key — decision unresolved | Medium | The ARCHITECTURE.md does not specify which key is the primary lookup. Must be decided before integration contracts are written. |
| Core Banking capability ownership questions open | Medium | Payment hierarchy config ownership (Core Banking vs. Onigiri Campaign Config), write-off approval flow, and interest calculation ownership are unresolved. Document decisions in ARCHITECTURE.md before feature work begins. |
| Event processing eventual consistency | Low | DaVinci is a read model — eventual consistency is acceptable for all use cases except real-time contact compliance. Contact compliance events must be near-real-time (< 30s p95). |
