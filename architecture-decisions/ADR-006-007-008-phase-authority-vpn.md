# ADR-006 — Phase-as-Product Model

**Status**: Accepted | **Date**: 2025-12 | **Author**: Salwan Mohamed

## Context
Platform engineering maturity models typically describe a linear progression implying all organizations should reach the highest level. This creates: organizations feeling "incomplete" if they don't advance, teams rushing to higher phases without stabilizing lower ones, and the model not accounting for organizations where Phase A is genuinely all that's needed.

## Decision
**Each phase is a complete, valuable product — not a step toward a higher phase. Advancement is optional and must be justified by organizational need.**

| Phase | Product Name | Value |
|-------|-------------|-------|
| A | Infrastructure Control as a Service | "Your infrastructure, under control, finally." |
| B | Explainable Infrastructure as a Service | "Know why before you ask." |
| C | Platform Knowledge as a Service | "Your platform remembers so you don't have to." |
| D | Governed Self-Service Platform | "Freedom within guardrails." |

## Consequences
**Positive**: Organizations can communicate what they have in product terms · No pressure to advance phases before ready · Phase A organizations are "complete," not "incomplete"  
**Negative**: Harder to create universal maturity benchmarks · Stakeholders may still push for Phase D without Phase A foundations

## Phase Advancement Criteria
Advancement from Phase N to N+1 requires:
1. All Phase N capabilities operational and stable
2. Phase N observability showing healthy signals
3. Explicit organizational need for Phase N+1
4. Human Owner authorization
5. ADR documenting the advancement decision

## Signals
- Sinai University Phase A operational with zero phase-skipping
- Stakeholders can explain their current phase value proposition
- No Phase B capabilities deployed before Phase A is stable

## Sinai University Status
| Phase | Status |
|-------|--------|
| A — Infrastructure Control | Active ✅ |
| B — Explainable Platform | Partial 🟡 |
| C — Platform Knowledge | Planned ⚪ |
| D — Governed Self-Service | Future ⚪ |


# ADR-007 — Human-Gated Decision Authority

**Status**: Accepted | **Date**: 2026-01 | **Author**: Salwan Mohamed

## Context
As AI tooling (Claude, Claude Code) integrates into platform workflows, there is pressure to extend AI authority from drafting → reviewing → approving → executing. The governance argument against this: systems that allow implicit approval erode over time. The human review step is not just a quality check — it maintains human accountability and organizational learning.

## Decision
**Human owners retain exclusive decision authority. AI tools draft and analyze; humans approve. No artifact is authoritative until a named human signs off.**

| Actor | Can Do | Cannot Do |
|-------|--------|-----------|
| Claude (AI) | Draft · Analyze · Identify risks · Generate artifacts | Approve decisions · Finalize ADRs · Define authority |
| Claude Code | Execute scripted tasks under PM direction | Self-authorize scope · Modify governance docs |
| Human Owner | Approve all P1/P2 decisions · Close ADRs · Authorize phase transitions | None — sole authority |

## Consequences
**Positive**: Governance integrity maintained · Organizational learning preserved · Accountability clear · Audit trail complete  
**Negative**: Slower decision velocity · Requires humans to remain engaged with platform governance

## Enforcement Mechanisms
1. DC Migration Platform: All ADRs require named human approver before merge to `main`
2. Platform OS: Explicit closure required for all decisions; no auto-acceptance
3. Nexus Core: Core ADRs require explicit status field (Proposed → Accepted/Rejected)

## Signals
- Zero ADRs in Accepted status without named human approver
- AI-generated drafts always marked as PROPOSED
- No phase transition executed without documented Human Owner authorization


# ADR-008 — Full-Mesh IPsec VPN Over Hub-and-Spoke

**Status**: Accepted | **Date**: 2025-12 | **Author**: Salwan Mohamed

## Context
Sinai University's 3 branch sites required secure inter-branch connectivity. Existing hub-and-spoke via central datacenter created: single point of failure for inter-branch traffic, all traffic routed through datacenter (latency), and datacenter maintenance windows silently killing inter-branch communication. Each branch already has an independent internet connection, making full-mesh technically viable.

## Decision
**Replace hub-and-spoke with full-mesh IPsec VPN — direct tunnels between all branch pairs.**

```
[Kantra]──IPsec──[Katamia]
    │                │
    └──IPsec──[Arish]──┘

Tunnel count: 3 total (N×(N-1)/2 for N=3 sites)
Hub dependency: ZERO
```

## Consequences
**Positive**: Datacenter outage does not affect inter-branch connectivity · Lower latency (direct path) · Each branch failure domain isolated · Simpler failure diagnosis  
**Negative**: Tunnel count grows quadratically with sites · Full-mesh impractical beyond ~7-8 sites

## Scale Threshold Analysis
| Sites | Tunnels | Recommendation |
|-------|---------|---------------|
| 3 | 3 | Full-mesh ✅ |
| 5 | 10 | Full-mesh ✅ |
| 8+ | 28+ | Hub-and-spoke or SD-WAN |

## Alternatives Considered
| Alternative | Rejected Because |
|-------------|-----------------|
| Hub-and-spoke | SPOF for inter-branch; datacenter dependency unacceptable |
| SD-WAN overlay | Cost; operational complexity; overkill for 3-site deployment |
| MPLS circuit | Cost; procurement timeline |

## Signals
- Inter-branch latency <50ms measured via ICMP probes
- Zero inter-branch outages caused by datacenter maintenance
- Tunnel uptime >99.5% measured via Prometheus Blackbox

## Critical Finding
Cairo-S2S tunnel: IKE Phase 1 UP but Phase 2 failing (1,135 attempts). Investigation needed: likely Phase 2 selector mismatch. This finding came from documentation, not monitoring.
