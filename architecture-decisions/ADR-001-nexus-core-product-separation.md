# ADR-001: Nexus Core/Product Separation Model

**Status:** Accepted  
**Date:** 2025-12-07  
**Decider:** Salwan Mohamed  
**Project:** Nexus Platform Engineering  

---

## Context

Building a platform for Sinai University required a system that could apply the same governance principles across multiple sites (3 campuses) with different physical infrastructure, different vendor constraints, and different operational maturity levels. The initial approach of a single monolithic repository quickly produced conflicts: environment-specific configuration was polluting architectural doctrine; vendor tool choices were being promoted to principles; and each site's operational team wanted to diverge without governance oversight.

The core question was: **how do you separate what is universally true about your platform from what is specific to where you deploy it?**

---

## Constraints

| Constraint | Impact |
|------------|--------|
| 3 campus sites with different network topologies | Single config model would require constant overrides |
| Limited platform team (1 principal + site ops) | Cannot maintain N diverging forks independently |
| University budget cycles | Vendor choices must be stable; can't refactor the core every procurement cycle |
| Junior site operators must be able to consume the platform | Doctrine cannot require deep architectural understanding |
| Decisions must be auditable for university governance | Every change needs a traceable owner and rationale |

---

## Decision

**Separate the platform into two distinct layers: a vendor-agnostic Core (defines principles, governance, capability contracts, and ADRs) and Product repositories (implement Core contracts with specific vendor choices for a specific environment).**

Core repository: `nexus-platform-engineering`  
Product repository: `nexus-platform-product-sinai-university`

Authority direction is one-way: **Core → Product**. Products cannot modify Core. If a Product ADR conflicts with a Core ADR, the Core wins.

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|----------------|
| **Single monorepo with environment folders** | Environment-specific config would pollute architectural doctrine; no clean governance boundary |
| **Full fork per environment** | Creates N independent codebases; governance and doctrine drift inevitable; no upgrade path |
| **Helm values per environment** | Solves config, not governance; doesn't address ADR authority or architectural principles |
| **Shared library + environment repos** | Closer, but doesn't define the authority model or what cannot be overridden |

---

## Trade-offs

**What we gained:**
- Architectural doctrine is stable and auditable; 183 ADRs define the Core
- Products are disposable — can replace the Sinai University implementation without touching Core
- New environments can consume Core directly without forking governance
- Site operators consume capability contracts, not architectural complexity

**What we gave up:**
- Initial overhead of defining the Core/Product boundary explicitly
- Product ADRs must reference Core ADRs — more documentation discipline required
- Cannot take a "quick shortcut" in a Product without acknowledging the governance debt

---

## Consequences

- Nexus Platform Core reached v14.0 with 183 ADRs in STEWARDSHIP MODE (architecturally complete)
- Sinai University Product has 14 implementation ADRs, each referencing the Core capability it satisfies
- A compliance checker script validates that Product ADRs don't contradict Core principles
- The model is now portable: any new organization can consume the Core and create their own Product

---

## Evidence

- [nexus-platform-engineering README](https://github.com/Salwan-Mohamed/nexus-platform-engineering) — Core/Product boundary definition
- [nexus-platform-product-sinai-university AUTHORITY.md](https://github.com/Salwan-Mohamed/nexus-platform-product-sinai-university/blob/main/AUTHORITY.md) — Authority model enforcement
- [Capability Mapping Table](https://github.com/Salwan-Mohamed/nexus-platform-product-sinai-university#capability-mapping) — 14 Core → Product capability implementations
- Core ADR-0135: Design-First Architecture Repository
- Core ADR-0091: Nexus Platform Philosophy

---

## Principal-Level Signal

> This decision separates **what your platform believes** from **what your platform runs on**. It is the difference between an infrastructure team and a platform engineering organization. Most teams never make this separation explicit — which is why their platforms become tightly coupled to vendor choices and cannot evolve without full rewrites.
