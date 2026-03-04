# Changelog: Atomic Architecture Migration

**Product**: Sensei (Branch Operations Orchestration)
**Portfolio**: Operations
**Changelog #**: 002
**Layer Affected**: Portfolio / Product / Capability
**Date**: 2026-03-04
**Author**: Claude (AI-assisted migration)

---

## What Changed

### Directory Restructure
- **Moved**: `product/sensei/` → `product/operations/sensei/`
- Git history fully preserved via `git mv` (all ATLAS.md, changelogs, and prototype/ directory tracked as renames)
- Working directory now lives under the `operations/` portfolio subdirectory in alignment with CLAUDE.md file system convention
- `product/sensei/prototype/` moved as-is to `product/operations/sensei/prototype/` — HTML prototype is a product artifact and stays with the product

### Files Created

| File | Layer | Description |
|------|-------|-------------|
| `product/operations/sensei/PRODUCT.md` | Product | Full product spec: problem statement, value proposition, IS/IS NOT boundary (emphasis: Sensei does NOT own contact compliance data — DaVinci owns it), 6-capability registry, task lifecycle state machine (Mermaid), integration map, KPIs |
| `product/operations/sensei/capabilities/playbook-engine/CAPABILITY.md` | Capability | HQ System Templates + Branch Variant fork model; compliance-locked steps (🔒); outcome-based transition routing; template version sync |
| `product/operations/sensei/capabilities/task-engine/CAPABILITY.md` | Capability | Unified task lifecycle (CREATED → ASSIGNED → ACTIVE → CLOSED + OVERDUE + ESCALATED); 4 task sources: playbook_step / event_rule / manual / external; TaskCreationRequest contract; TaskCompleted feedback events |
| `product/operations/sensei/capabilities/work-queue/CAPABILITY.md` | Capability | Grouped action buckets (Calls/Visits/Admin); priority sub-groups (Overdue > High DPD > Normal); one-by-one primary mode; rapid-fire extended mode |
| `product/operations/sensei/capabilities/performance-dashboard/CAPABILITY.md` | Capability | Supervisor view: team workload, exception panel (5 alert types), daily scorecard; Staff view: personal metrics, monthly objectives, gamified leaderboard |
| `product/operations/sensei/capabilities/contact-compliance/CAPABILITY.md` | Capability | Subscribes to DaVinci compliance events; auto-skips tasks on ContactLimitReached; warning badge on ContactLimitApproaching; blocks tasks on ContactWindowClosed. **BOUNDARY: Does NOT own contact log or frequency limits — DaVinci owns those.** |
| `product/operations/sensei/capabilities/template-library/CAPABILITY.md` | Capability | HQ-owned action type definitions; typed outcomes per action; required fields per outcome (e.g., PTP → amount + date); SLA defaults; escalation rules |
| `product/operations/sensei/artifacts/diagrams/.gitkeep` | Infrastructure | Scaffold: artifacts/diagrams/ directory (was missing entirely) |
| `product/operations/sensei/artifacts/research/.gitkeep` | Infrastructure | Scaffold: artifacts/research/ directory (was missing entirely) |

### Files Kept (Unchanged)
- `product/operations/sensei/ATLAS.md` — primary source of truth
- `product/operations/sensei/changelogs/CHANGELOG_001_initial_product_definition.md` — preserved as-is
- `product/operations/sensei/prototype/` — HTML prototype preserved as product artifact

---

## Rationale

Sensei orchestrates branch operations for COs — it is the day-to-day task executor that drives compliance with collection playbooks, action verification against 3CX, and performance visibility for supervisors. As an operational orchestration tool for field staff, it belongs in the `operations/` portfolio alongside Matcha (verification engine) and Wasabi (AI verification layer), all three sharing DaVinci as their master data backbone.

---

## Decision Log

| Decision | Rationale |
|----------|-----------|
| Portfolio: Operations | Sensei is a branch operational tool (not Credit origination, not Platform data infrastructure). Operations portfolio is the correct home. |
| Contact compliance boundary documented at both PRODUCT and CAPABILITY layers | This boundary (DaVinci owns contact log; Sensei is a consumer of events) is the most commonly misunderstood cross-product boundary in the system. Redundant documentation is intentional. |
| ATLAS.md kept in-place | Source of truth during transition |
| FEATURE_*.md deferred | Requires engineering owner assignment and testable acceptance criteria |
| prototype/ moves with product | HTML prototype is a product artifact (historical reference for UX decisions), not infrastructure. Stays under operations/sensei/ |
| DaVinci/Sensei resolution ownership split documented | DaVinci owns ResolutionRequest lifecycle state; Sensei only creates and completes tasks. This split is documented in both PRODUCT.md and data-resolution-workflow CAPABILITY.md to prevent logic leakage. |

---

## Links

- [PRODUCT.md](../PRODUCT.md)
- [PORTFOLIO.md](../../PORTFOLIO.md)
- [PORTFOLIO_CATALOG.md](../../../PORTFOLIO_CATALOG.md)
- [ATLAS.md](../ATLAS.md) (source)
