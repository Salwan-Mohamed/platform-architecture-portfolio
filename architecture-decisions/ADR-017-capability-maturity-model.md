# ADR-017: Capability Maturity Model

**Status:** Accepted  
**Date:** 2026-06-09  
**Source:** ARCH-0001 (Capability Maturity Model, 2026-05-07)  
**Project:** Sinai University Platform — Platform Governance Layer

---

## Context

ADR-012 (Evidence-Gated Phase Transitions) established that phase advancement requires documented evidence satisfying gate criteria. That model operates at the phase level (Phase A → B → C → D). It does not address capability-level granularity: within a phase, individual capabilities (FortiGate policy automation, Terraform provisioning, AWX runbook execution, storage lifecycle) progress through different maturity states at different rates.

Without a capability-level maturity model, write-capable operations can be enabled against capabilities that have not yet demonstrated observability, backup integrity, or inventory completeness. The result is write-capability activated on the basis of tool deployment, not operational readiness.

ARCH-0001 formalises the Capability Maturity Model that resolves this: a 7-level progression binding write-capability activation to demonstrated operational readiness, not to calendar or phase advancement.

## Constraints

- Platform manages infrastructure across 3 sites with a 4-person team — write-capability errors are high-cost to recover
- Different capabilities have fundamentally different risk profiles (DNS update vs. firewall policy automation vs. SAN zoning change)
- Human Owner must retain approval authority over any write-capable activation — no automated advancement into write-capable levels
- Assessment must be event-driven and evidence-based, not calendar-based — advancing on schedule without evidence is governance theater
- The model must apply consistently across all platform capabilities, not be capability-specific

## Decision

**Adopt a 7-level capability maturity progression as the canonical model for every capability in the Sinai University Platform. Write-capability is gated to maturity level — not to phase, not to tool deployment, not to calendar.**

The 7 levels:

| Level | Name | Description |
|---|---|---|
| 1 | disabled | Capability not active. Platform does not interact with underlying systems beyond declaring capability existence. |
| 2 | observed | Read-only. Platform collects observability signals (metrics, logs, flow data) without taking any action. |
| 3 | backup-enabled | Capability state backed up at known cadence with verified integrity; recovery validated. |
| 4 | read-only-inventory | Capability state enumerated into structured inventory; platform reads systematically beyond observability signals but performs no writes. |
| 5 | safe-write | Accepts low-risk, reversible writes. Lowest write-capable level. **Requires Human Owner approval to activate.** |
| 6 | controlled-write | Accepts higher-risk writes under controlled-change discipline (approval gates, evidence packs, recovery plans). **Requires Human Owner approval.** |
| 7 | full-managed | Fully managed by platform under strictest activation criteria. **Requires: successful controlled-write operation + stable operation evidence + recovery/rollback validation + explicit Human Owner approval.** |

Levels 1–4 are read-only or non-write states. Levels 5–7 are write-capable. No capability enters a write-capable level without satisfying the exit criteria below.

**Exit criteria for any level advancement:**
- Evidence pack exists for the current level
- Validation evidence exists for the target level
- Rollback or recovery path is known (where applicable)
- Operational risk impact is understood
- Required owners have reviewed the change
- Any transition into a write-capable level (5, 6, or 7) requires explicit Human Owner approval

**Ownership model:**
- Platform/operator roles: collect evidence and recommend advancement
- Platform Architect: validate readiness
- Human Owner: final approval for write-capable transitions (Levels 5, 6, 7)

**Assessment cadence** — event-driven, not calendar-driven. Mandatory review at:
- Evidence milestones
- Wave gates
- Before customer onboarding
- Before enabling any write-capable operation
- Quarterly minimum for production-facing capabilities

## Alternatives Considered

**Phase-level gates only (existing ADR-012 model)** — advance capabilities when the phase advances. Rejected: phase advancement is a platform-wide gate; individual capabilities within a phase can be at very different operational readiness levels. A platform in Phase B may have Prometheus at Level 7 (full operational observability) and FortiGate policy automation at Level 2 (observed only). Conflating the two produces risk.

**Binary ready/not-ready per capability** — capability is either safe to automate or it isn't. Rejected: the progression from observed to backup-enabled to read-only-inventory is operationally meaningful. Each level de-risks the next. Skipping to a binary model removes the intermediate evidence requirements that make write-capability defensible.

**Self-certification by engineering team** — team declares a capability write-ready without formal gate. Rejected: same problem as calendar-based advancement. Self-certification has no evidence requirement and no second review. Given the 4-person team and multi-site infrastructure, unreviewed write-capability on production systems is an unacceptable risk posture.

## Trade-offs

**Accepted:** Additional governance overhead per capability advancement. Every write-capable transition requires evidence collection, owner review, and Human Owner approval. This adds time. At a small team operating 3-site infrastructure, that time is far less than the recovery cost of a write-capability error on an under-observed system.

**Accepted:** Capabilities at Level 2 (observed) cannot be automated even if the tooling exists. Tooling availability is not the gate — operational readiness is. This means some automation that could be technically deployed will be deferred until the capability reaches the appropriate level.

**Gained:** Write-capability on any platform capability is provably grounded in evidence. Every automated operation against production infrastructure can be traced to a level assessment, an evidence pack, and a Human Owner approval decision. That audit trail is what transforms platform automation from an operational risk into a governance asset.

## Consequences

- All platform capabilities are tracked against the 7-level model in the platform governance repository
- No write-capable automation is activated without a level advancement approval event on record
- ADR-012 (phase-level gates) and this ADR (capability-level gates) operate in parallel — both must be satisfied
- The model applies retroactively: capabilities currently operating at a write-capable level are grandfathered at their current level; advancement from the grandfathered level requires applying exit criteria forward
- This model is the authority for downstream architecture decisions about Infrastructure Automation activation (ADR-018)

## Evidence

- ARCH-0001 approved 2026-05-07 with Human Owner Decision Payload verbatim
- ADR-012 (evidence-gated phase transitions) established the foundational governance posture; this ADR extends it to capability granularity
- Phase C EXECUTION-TRACKER.md provides the current capability state baseline for grandfathering

## Principal-Level Signal

Write-capability is not a deployment milestone. Deploying Terraform does not mean Terraform should be running `apply` against production. Deploying AWX does not mean AWX should be executing runbooks against production network devices. The maturity model makes the distinction explicit: a capability at Level 2 (observed) is deployed and visible. A capability at Level 5 (safe-write) has been observed, backed up, inventoried, and assessed as ready for reversible writes — with Human Owner sign-off. The distance between Level 2 and Level 5 is not time. It is evidence.
