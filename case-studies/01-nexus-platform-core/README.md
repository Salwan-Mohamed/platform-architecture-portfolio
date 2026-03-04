# Case Study 01 — Nexus Platform Engineering Core

> *Building a vendor-agnostic governance foundation that any organization can implement*

**Repository**: [nexus-platform-engineering](https://github.com/Salwan-Mohamed/nexus-platform-engineering) (private)  
**Status**: Architecturally complete — STEWARDSHIP MODE (v14.0.2)  
**Scale**: 183 ADRs · 4 phases · 11 source chapters · 8 observability axes

---

## Problem

Most platform engineering projects fail the same way: they build excellent infrastructure but embed all governance in the tool choices. When the tool changes, the governance evaporates. Engineers copy configurations between environments without understanding the decisions behind them.

The deeper problem: **platforms are built by people who will eventually leave**. What remains after they leave determines whether the platform survives or degrades.

---

## Environment

- Multi-site organization (university environment as primary consumer)
- Geographically distributed infrastructure across 3+ branches
- Mixed vendor landscape: VMware, FortiGate, Aruba, various Kubernetes distributions
- Small platform engineering team
- Strong need for governance that survives personnel changes

---

## Constraints

- **No tool lock-in**: Core must work regardless of whether the consumer uses VMware or bare metal, FortiGate or pfSense, ArgoCD or Flux
- **Progressive adoption**: Organizations should be able to start at Phase A and never advance — advancement is optional, not assumed
- **Human authority preserved**: AI and automation are operators, not authorities
- **Evidence over assertion**: No ADR is valid without signal reference

---

## Architecture Decision

### Core/Product Separation

The foundational architectural decision: **separate what the platform IS from what it's built WITH**.

```
CORE (nexus-platform-engineering)
  ├── Defines: principles, governance models, authority frameworks
  ├── Defines: phase definitions, capability contracts
  └── Does NOT contain: tool names, vendor choices, config files

PRODUCT (nexus-platform-product-sinai-university)
  ├── Implements: capability contracts with specific vendors
  ├── Contains: FortiGate, RKE2, ArgoCD, Keycloak configurations
  ├── References: Core ADRs as authority for all decisions
  └── Is disposable: can be replaced without affecting Core
```

**Critical insight**: governance documents must be more durable than the tools they govern.

### Phase-as-Product Model

| Phase | Product Name | Philosophy |
|-------|-------------|------------|
| A | Infrastructure Control as a Service | *"Your infrastructure, under control, finally."* |
| B | Explainable Infrastructure as a Service | *"Know why before you ask."* |
| C | Platform Knowledge as a Service | *"Your platform remembers so you don't have to."* |
| D | Governed Self-Service Platform | *"Freedom within guardrails."* |

Advancement between phases is **optional and explicit** — not assumed, not automatic.

### 183 ADRs Across 11 Domains

Every significant architectural decision captured with: Context · Decision · Consequences · Authority Model · Phase Requirements

Domains covered: Governance · Observability · Security · Pipeline/Deployment · Scaling · Organizational · FinOps · Capabilities · Network · Infrastructure

---

## Trade-offs

| Trade-off | What Was Gained | What Was Given Up |
|-----------|----------------|-------------------|
| Agnostic Core adds abstraction | Reusability across environments | Documentation overhead per implementation |
| 183 ADRs requires maintenance discipline | Complete architectural memory | Ongoing curation burden |
| Human-only decision authority | Governance integrity | Speed of some automated workflows |
| Phase-optional model | Adoption at any maturity level | Pressure to advance to "higher" phases |

---

## Outcome

- **v14.0.2** — 183 ADRs, architecturally complete
- Core declared in **STEWARDSHIP MODE** — frozen for governance, products drive evolution
- Sinai University successfully implementing as first product consumer
- Core/Product separation proven: product can be replaced without Core modification

### What I Would Do Differently

1. **Start with fewer ADRs, evolve more**: 183 ADRs written somewhat speculatively. A leaner Core (50 ADRs) with products driving expansion might have been more grounded.
2. **Signal registry before governance**: Define what signals tell you the platform is working before defining governance rules.
3. **Public Core earlier**: The value of a public vendor-agnostic core is peer review and external challenge to architectural assumptions.

---

## Operational Lessons

1. **Documentation IS engineering work** — ADR writing should be scoped, estimated, and tracked like implementation work
2. **Core freeze is a feature** — declaring a Core frozen enables product teams to build confidently without governance churn
3. **Authority models must be explicit before automation** — "who can approve what" is more important than any specific tool choice
4. **Phase labels communicate expectations** — calling Phase A "Infrastructure Control as a Service" tells stakeholders exactly what they're getting

---

## Key ADRs Referenced

| ADR | Title | Significance |
|-----|-------|--------------|
| 0091 | Platform Philosophy | Nexus is an organizational coordination system, not a product |
| 0114 | Phase-Oriented Product Model | Phase advancement is optional |
| 0126 | No Implicit Authority | All authority must be explicit and traceable |
| 0135 | Design-First Architecture | Governance before implementation |
| 0139 | Vendor-Neutral Architecture | No tool names in Core |
| 0142 | Intent Contracts Before Automation | Define what before automating how |
| 0088 | AI Is Not an Actor | AI operates within authority, does not define it |
