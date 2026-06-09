# ADR-018: Infrastructure Automation Positioning

**Status:** Accepted  
**Date:** 2026-06-09  
**Source:** ARCH-0002 (Infrastructure Automation Positioning, 2026-05-08)  
**Project:** Sinai University Platform — Platform Governance Layer

---

## Context

ADR-016 (Infrastructure Execution Stack) established the tools: Terraform for provisioning, AWX for runbook automation, Tekton for CI, Sealed Secrets for credential management. The tool selection is settled. What is not formalised is the architectural relationship between Infrastructure Automation and the Core Platform — specifically: does the Core Platform depend on Infrastructure Automation to operate?

This question matters because the Sinai University platform manages 3-site infrastructure with a 4-person team. If a Terraform run fails or AWX is unavailable, existing workloads — student authentication via Keycloak, ArgoCD reconciliation, Grafana dashboards, Cilium network policies — must continue to operate without degradation. Infrastructure Automation is a change mechanism, not a runtime dependency.

ARCH-0002 formalises this positioning: Infrastructure Automation is a Capability Layer Extension, not a Core Platform runtime dependency, with concrete failure-mode commitments per named component.

## Constraints

- AWX is explicitly deferred (IMPL-0030 §2.4, Phase C DEFERRALS.md §5) — Terraform + Ansible are the current execution stack
- The Core/IA dependency rule must be verifiable — named components, testable failure modes
- Visibility must be established at the domain/function level before write-capability is activated — different domains have different risk profiles
- High-risk domains (firewall policy automation, SAN zoning, core switching) require controlled-write maturity and Human Owner approval, not just safe-write
- The positioning must not pre-decide the v2.0 layer topology — functional definition is stable; topology is deferred to v2.0 baseline candidate drafting

## Decision

**Infrastructure Automation is a Capability Layer Extension. Core Platform must not depend on Infrastructure Automation. Concrete failure-mode commitments bind this principle. Visibility-first is mandatory at the relevant domain/function boundary before write-capable activation.**

**Core/IA dependency rule — named failure-mode commitments:**

If Infrastructure Automation (Terraform, Ansible, AWX) is unavailable, disabled, degraded, or removed:
- RKE2 control plane and worker runtime must continue to operate
- ArgoCD GitOps reconciliation for existing Applications must continue to operate
- Keycloak identity services must continue to authenticate and authorise
- Cilium networking and Gateway API exposure must continue to operate
- Prometheus/Grafana/Loki/Robusta observability must continue to collect and display
- Existing workloads must continue to run
- Read-path operations for deployed workloads must not depend on IA availability

These are not aspirational commitments — they are architectural constraints that the platform deployment must satisfy at all times. IA availability is irrelevant to Core Platform survival.

**Infrastructure Automation Modes** are bound to ADR-017 Capability Maturity Model levels:

| Mode | Maturity Level | Description |
|---|---|---|
| observed | Level 2 | Read-only signal collection |
| backup-enabled | Level 3 | State backed up, recovery validated |
| read-only-inventory | Level 4 | Systematic inventory, no writes |
| safe-write | Level 5 | Low-risk, reversible writes. Human Owner approval required. |
| controlled-write | Level 6 | Higher-risk writes under controlled-change discipline. Human Owner approval required. |
| full-managed | Level 7 | Full management under strictest criteria + stable operation evidence + Human Owner approval. |

**Visibility-first principle — per domain/function, not global:**

Before any write-capable mode is activated against a target domain, that domain must demonstrate:
- Current-state visibility or inventory
- Backup/export evidence where applicable
- Pre-change validation method
- Post-change validation method
- Rollback or recovery path
- Ownership boundary clarity
- Approval proportional to risk

This gate is **per domain/function**, not a global all-infrastructure blocker. Lower-risk functions (tightly-scoped DNS/DHCP updates, low-risk object creation) may reach safe-write while high-risk domains (SAN zoning, firewall policy automation, core switching, storage-path changes) remain at read-only-inventory. Risk profile governs the pace, not a uniform gate.

**High-risk domain additional requirements:** controlled-write maturity (Level 6) + explicit Human Owner approval + rollback/validation evidence stronger than baseline. Safe-write is not sufficient for high-risk domains.

## Alternatives Considered

**Core Platform optionally depends on IA** — some Core components use IA for day-2 operations (e.g., ArgoCD uses Terraform for cluster provisioning). Rejected: this makes Core Platform recovery dependent on IA availability during an incident. The exact scenario where IA is most likely to be unavailable (infra incident) is the same scenario where Core Platform availability is most critical.

**Global visibility-first gate** — no write-capable automation until full infrastructure visibility is established. Rejected: over-rigid. Tightly-scoped DNS updates do not need SAN storage to be fully inventoried before proceeding. The relevant-domain gate achieves the same safety objective with less operational friction.

**Single write-capability tier** — safe-write only, no controlled-write distinction. Rejected: firewall policy automation carries materially different risk than a DNS record update. A single tier either over-gates low-risk operations or under-gates high-risk ones. The two-tier model (safe-write / controlled-write) is grounded in the different blast radius and reversibility characteristics of the operations.

## Trade-offs

**Accepted:** High-risk domain automation is harder to activate. SAN zoning changes, firewall policy automation, core switching changes all require controlled-write maturity + Human Owner approval. These are not quick to enable. For a 3-site, 4-person team managing production infrastructure, that activation cost is the correct operational posture.

**Accepted:** Functional definition without topology commitment. The v2.0 layer model (where Infrastructure Automation sits in the architecture diagram) is deferred. This ADR establishes what IA does and what it cannot affect, without locking down its topological position in the architecture. That deferral is intentional — the v2.0 baseline candidate will resolve the topology.

**Gained:** The Core/IA dependency rule is testable. "Does RKE2 continue to operate if Terraform is unreachable?" is a verification test. These commitments can be validated in a controlled failure exercise, not just asserted. That testability is what makes this a governance commitment rather than a design aspiration.

## Consequences

- Every Infrastructure Automation deployment plan must satisfy the §3.3 failure-mode commitments before activation
- Mode activation (safe-write, controlled-write, full-managed) requires ADR-017 exit criteria + visibility-first evidence gates — both must be satisfied
- AWX remains deferred; Terraform + Ansible are the current execution stack
- High-risk domains (SAN, firewall policy, core switching) cannot enter safe-write — they require controlled-write maturity
- This ADR enriches ADR-016 (execution stack tools) with the governance positioning that determines when those tools are permitted to write
- Platform v2.0 layer topology is forward-pointed — this ADR's functional definition is stable; topological placement is v2.0 scope

## Evidence

- ARCH-0002 approved 2026-05-08 with Human Owner Decision Payload verbatim
- IMPL-0030 §2.4 confirms AWX deferral; Phase C DEFERRALS.md §5 reaffirms it
- Phase A TARGET-STATE-ARCHITECTURE.md §2.2 names the Core components referenced in failure-mode commitments
- ADR-017 (Capability Maturity Model) provides the level definitions that Modes bind to

## Principal-Level Signal

The most dangerous moment in platform automation is not when something breaks — it is when a platform engineer assumes that deploying an automation tool means it is safe to use that tool against production. Infrastructure Automation positioning is the architectural answer to that assumption: deployment is not activation, activation requires evidence, and Core Platform survival must never depend on automation being available. Automation is an accelerator for change. It must never become a dependency for existence.
