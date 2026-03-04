# ADR-001: Core/Product Separation Model

**Status:** Accepted  
**Date:** 2026-01-23  
**Decider:** Salwan Mohamed  
**Domain:** Governance  
**Project:** Nexus Platform Engineering

---

## Context

Building a platform for Sinai University (3 campuses, multi-vendor environment) created an immediate tension: every architecture decision was both universal ("GitOps is better than manual ops") and environment-specific ("FortiGate 1101E is the firewall at Kantra site").

If these two types of decisions live in the same repository, one of two failure modes occurs:
1. **Doctrine becomes polluted with vendor details** — the platform becomes non-portable and future environments can't benefit from the thinking
2. **Implementation details get lost in abstraction** — operators can't find what they need to actually run the environment

The question: how do you maintain architectural integrity across both levels simultaneously?

---

## Constraints

- Single engineer initially — governance overhead must be sustainable
- University budget — vendor choices are constrained by licensing and existing investments
- Operational team with varying technical maturity — documentation must be findable and actionable
- Need to eventually support multiple environments — architecture must be reusable without copy-paste

---

## Decision

**Separate the platform into two distinct repository types: a vendor-agnostic Core (principles + governance + ADRs) and environment-specific Product repositories (vendor choices + configurations + implementation ADRs), with a strict one-way authority relationship where Core always wins.**

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|-----------------|
| Single monolithic repository | Doctrine gets polluted with vendor specifics; non-portable; impossible to reuse for future environments |
| Separate repos with no formal relationship | Implementations drift from principles; no traceability; governance becomes performative |
| Fork-per-environment model | Forks diverge; principle changes don't propagate; creates N copies of the same thinking |
| Wiki-based documentation | No versioning; no change control; no authority model; becomes stale immediately |

---

## Trade-offs

**Given up:** Simplicity. A single repo is easier to navigate when you know where everything is.

**Gained:**
- Nexus Core can evolve independently of any specific vendor environment
- Sinai University Product can be replaced entirely (different vendor stack) without changing platform doctrine
- Other environments (future) can implement the Core without inheriting Sinai-specific configurations
- Compliance checking is scriptable: `core-compliance-checker.sh` validates that Product ADRs reference Core ADRs

---

## Consequences

**Positive:**
- Nexus Platform Core reached 183 ADRs in STEWARDSHIP mode — architecturally complete
- Sinai University Product has 14 Implementation ADRs, all referencing Core
- The authority model is explicit: `if this repository conflicts with the core, the core wins`
- Future environments can be created as new Product repositories consuming the same Core

**Negative:**
- Cognitive overhead of navigating two repositories
- Contributors must understand the Core/Product boundary before making changes
- Risk of Product ADRs being written without Core references (mitigated by compliance scripts)

---

## Evidence

- [nexus-platform-engineering](https://github.com/Salwan-Mohamed/nexus-platform-engineering) — Core repository: 183 ADRs, vendor-agnostic
- [nexus-platform-product-sinai-university](https://github.com/Salwan-Mohamed/nexus-platform-product-sinai-university) — Product: 14 implementation ADRs
- [AUTHORITY.md](https://github.com/Salwan-Mohamed/nexus-platform-product-sinai-university/blob/main/AUTHORITY.md) — Explicit authority model
- Compliance checker script: `scripts/core-compliance-checker.sh`

---

## Operational Lesson

The Core/Product separation paid off immediately when the FortiGate implementation ADR needed to be revised. The revision only affected `nexus-platform-product-sinai-university` — the Core doctrine ("Firewall is a platform capability contract") was unchanged. Without the separation, a vendor-specific change would have required validating its impact on all platform principles.

---

*This ADR documents a decision made at the architectural foundation of the Nexus Platform. It is the governance primitive from which all other decisions derive.*
