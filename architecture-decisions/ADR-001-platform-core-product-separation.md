# ADR-001 — Platform Core / Product Separation

**Status**: Accepted  
**Date**: 2025-12  
**Author**: Salwan Mohamed  
**Context**: Nexus Platform Engineering

---

## Context

Building a platform governance system that needs to survive tool changes without losing architectural memory, be reusable across different environments, give product teams clear authority boundaries, and prevent governance drift when products evolve.

The naive approach — one repo containing everything — creates a coupling problem: when the tool changes, the principle documentation changes too, making it impossible to distinguish between "we changed our thinking" and "we changed our tools."

---

## Decision

**Separate the governance foundation (Core) from the implementation (Product) into distinct repositories with a defined authority relationship.**

```
Core Repository
  - Vendor-agnostic
  - Defines: principles, governance models, capability contracts
  - Authority: immutable from product perspective
  - Evolution: slow, deliberate, versioned

Product Repository
  - Vendor-specific
  - Implements: capability contracts with chosen tools
  - Authority: subordinate to Core
  - Evolution: fast, environment-driven

Authority rule: If Product conflicts with Core, Core wins.
```

---

## Consequences

### Positive
- Core ADRs survive tool migrations — when VMware is replaced, Core doesn't change
- Multiple Product implementations can share one Core — reusability without forking
- Clear conflict resolution — "which repo wins?" has a defined answer
- Product teams have autonomy within Core constraints

### Negative
- Two repos to maintain instead of one
- Product teams must learn Core before making implementation decisions
- Core versioning discipline required — products must track which Core version they implement

---

## Alternatives Considered

| Alternative | Rejected Because |
|-------------|------------------|
| Single monorepo | Coupling between principles and implementations |
| Templates/forks | Governance divergence over time; no authority model between template and fork |
| Core as package dependency | Too rigid; governance documents are not software dependencies |

---

## Signals

- Products can change vendors without modifying Core ADRs
- Compliance checker passes on every product ADR merge
- Core version remains stable across multiple product releases

---

## Implementation Evidence

- `nexus-platform-engineering` — Core repository (v14.0.2, 183 ADRs)
- `nexus-platform-product-sinai-university` — Product implementation (Core v13.0, 127 ADRs referenced)
- `scripts/core-compliance-checker.sh` — validates product alignment with Core
