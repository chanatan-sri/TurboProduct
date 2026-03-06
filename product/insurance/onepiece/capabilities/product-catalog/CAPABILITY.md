# Capability: Product Catalog & Channel Configuration

> **Parent Product:** OnePiece (Insurance Distribution Platform)
> **Product Owner:** TBD
> **Status:** Draft
> **Last Updated:** 2026-03-05

---

## Business Function

Manages the master configuration of all insurance products, insurer partnerships, channel availability rules, payment eligibility, and policy issuance method mappings. This capability is the single source of truth for "what can be sold, where, how it's paid, and how the policy is issued."

---

## Feature Inventory

| # | Feature | Status | Description |
|---|---------|--------|-------------|
| 1 | Insurance Product Registry | Concept | Master list of insurance products (CMI, VMI types) with attributes |
| 2 | Insurer-Product Mapping | Concept | Configuration of which insurers offer which products, by coverage year, with available packages |
| 3 | Channel Availability Rules | Concept | Rules governing which products are available on which sale channels, including coverage year restrictions |
| 4 | Payment Eligibility Matrix | Concept | Configuration of payment terms and payment channels per sale channel |
| 5 | Issuance Method Configuration | Concept | Per insurer-product mapping of policy issuance method (file transfer, API, manual) |

---

## Business Rules

| Rule ID | Rule | Condition | Result |
|---------|------|-----------|--------|
| PC-001 | Motorcycle products are branch-only | Product in {VMI-MC-3, VMI-MC-5} AND Channel = Online | Block: product not available |
| PC-002 | Cash and Bill Payment QR are branch-only | Channel = Online AND Payment Channel in {Cash, Bill Payment QR} | Block: payment channel not available |
| PC-003 | Online is full payment only | Channel = Online | Installment not offered |
| PC-003a | 2C2P CC and IRR are full-payment only | Payment Term = Installment AND Payment Channel in {2C2P Credit Card, 2C2P IRR} | Block: payment channel not available |
| PC-004 | Coverage > 1 year is branch-only | Coverage Year > 1 AND Channel = Online | Block: not available online |
| PC-005 | Issuance method determined by insurer-product-coverage combination | Insurer = X AND Product = Y AND Coverage = Z | Return configured issuance method (REST API or Manual) |
| PC-006 | Manual issuance products are branch-only | Issuance Method = Manual AND Channel = Online | Block: not available online |
| PC-007 | Installment requires loan approval | Payment Term = Installment | Trigger loan approval flow |

---

## Open Questions

- What is the data model for insurer partnership management (contract terms, commission rates)?
- How frequently do insurer package offerings change? Real-time sync or periodic batch?
- Are there seasonal or campaign-based product availability overrides?
