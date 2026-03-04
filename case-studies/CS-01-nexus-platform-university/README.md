# Case Study 01: Building a Platform Architecture for Sinai University

**Type:** Platform Architecture + Governance Design  
**Scale:** 3 campuses, 1 datacenter, multi-vendor  
**Duration:** 2025 – 2026 (ongoing)  
**Role:** Platform Architect + Lead Engineer  
**Key Repos:** `nexus-platform-engineering`, `nexus-platform-product-sinai-university`, `university-network-architecture`

---

## The Problem

Sinai University was running infrastructure the way most institutions run infrastructure: by tribal knowledge, manual processes, and heroics. Three campuses with no shared understanding of what "the platform" actually was. Engineers fixing problems without documenting decisions. The next person to touch a system would start from zero.

The surface-level ask was: "Build us a Kubernetes platform." The real problem was deeper: the university had no way to make infrastructure decisions that would still make sense in two years. No governance. No authority model. No way to know what changed, when, or why.

---

## The Environment

| Dimension | Details |
|-----------|---------|
| Sites | 3 campuses (Kantra, Katamia, Arish) + datacenter |
| Network | Full-mesh IPsec VPN, FortiGate 1101E at each site |
| Firewall policies | 67 policies at Kantra, 34 interfaces, 5 VPN tunnels |
| Kubernetes | RKE2 production clusters via Cluster API (CAPV) |
| Virtualization | VMware vSphere |
| Identity | Active Directory → Keycloak OIDC broker |
| GitOps | ArgoCD reconciling all platform state |
| Team size | 1-2 platform engineers |
| ADRs written | 183 (Core) + 14 (Sinai Implementation) = 197 total |

---

## The Constraints

1. **Budget** — University budget means no managed cloud, no expensive enterprise platforms. Everything must be self-managed or open source.
2. **Team size** — 1-2 engineers means operational overhead must be minimal. The platform must operate mostly without manual intervention.
3. **Organizational maturity** — The teams consuming the platform have varied technical sophistication. The platform must be usable by non-expert consumers.
4. **Audit requirements** — University governance requires audit trails for infrastructure changes.
5. **Multi-site independence** — Campus operations cannot depend on datacenter availability. If the datacenter is down, campuses must still function.

---

## The Architecture

### Layer 1: Network Foundation

Full-mesh IPsec VPN between all three campus edge firewalls. No hub-spoke. No datacenter dependency for inter-campus traffic.

The design decision here was organizational, not technical: *branch autonomy is not a performance optimization, it's a resilience requirement.* A university that can't let students communicate between campuses because the datacenter is in maintenance has failed its operational mission.

```
                 KANTRA-SITE
                 FortiGate 1101E
                 │         │
          IPsec  │         │  IPsec
                 │         │
    ARISH-SITE ──┘         └── KATAMIA-SITE
    FortiGate       IPsec       FortiGate
         └─────────────────────────┘

No hub. No single point of failure.
Each branch has local internet breakout.
```

### Layer 2: Virtualization and Compute

VMware vSphere as the existing virtualization layer. Decision not to replace it: the cost and disruption of migrating away from vSphere exceeds the benefit at current scale. Use it, integrate with it, plan for future evolution.

Cluster API Provider vSphere (CAPV) for Kubernetes cluster lifecycle management. Clusters are code. Cluster upgrades are PRs.

### Layer 3: Kubernetes Platform

RKE2 for security defaults (FIPS, CIS Benchmark). Cilium for eBPF-based network policy — Layer 7 visibility without service mesh complexity. ArgoCD for continuous reconciliation.

The GitOps model means: **cluster state = Git state**. If you can't find it in a repository, it doesn't exist in the platform.

### Layer 4: Identity and Security

Active Directory is the existing identity authority (university-wide). Rather than replacing it, we introduced Keycloak as an OIDC broker — a translation layer between the legacy AD model and the modern OIDC model that Kubernetes and all platform tools expect.

The identity architecture decision: *don't fight the existing identity authority, bridge it.*

### Layer 5: Governance (the layer most platforms forget)

The Nexus Platform Core: 183 ADRs defining vendor-agnostic governance principles. The Core is in stewardship mode — architecturally complete, frozen for changes without exceptional justification.

The Sinai University Product: 14 implementation ADRs, each referencing a Core ADR, each documenting the specific vendor/tool choice and why.

The authority model: **Core → Product (one-way)**. If the Product conflicts with the Core, the Core wins.

---

## The Key Decisions

### Decision 1: Separate Core from Product
Documented in [ADR-001](../../architecture-decisions/ADR-001-nexus-core-product-separation.md).

Instead of one monolithic "platform repo," the Core defines vendor-agnostic principles and the Product implements them with specific vendor choices. This means the platform doctrine (183 ADRs) is reusable for any future environment.

**Why this matters for Principal-level engineers:** This is the decision that separates "I built a Kubernetes platform" from "I designed a platform architecture system."

### Decision 2: Full-Mesh VPN, Not Hub-Spoke
Documented in [ADR-007](../../architecture-decisions/ADR-007-full-mesh-vpn-over-hub-spoke.md).

Hub-spoke creates datacenter dependency. Full-mesh gives each branch autonomous connectivity. The trade-off is quadratic tunnel growth — acceptable at 3 sites, would need SD-WAN at 10+ sites.

### Decision 3: ArgoCD as the Platform Control Surface
Documented in [ADR-002](../../architecture-decisions/ADR-002-gitops-as-control-plane.md).

All cluster state is declared in Git. ArgoCD continuously reconciles. Drift is visible, alertable, and recoverable.

### Decision 4: Phase A Before Phase B
Documented in [ADR-005](../../architecture-decisions/ADR-005-phase-based-platform-maturity.md).

Phase A (Infrastructure Control) must be stable before Phase B (Explainable Platform) begins. The temptation to add capabilities before foundational ones are solid is the most common cause of platform failures.

---

## The Trade-offs

| What I gave up | What I gained |
|----------------|---------------|
| Speed to full capability | Each phase delivers standalone value |
| Single-repo simplicity | Core is portable, reusable, and doctrine-stable |
| Hub-spoke centralization | Branch autonomy and datacenter-independent resilience |
| Vendor diversity | Single skills requirement, unified management plane |
| Flexibility to skip governance | Audit trail, knowledge transfer, organizational trust |

---

## The Outcome

- **183 ADRs** in Nexus Platform Core — architecturally complete, in stewardship mode
- **14 implementation ADRs** for Sinai University, all referencing Core
- **Phase A active** at Sinai University: Infrastructure Control operational
- **3-site network** with full-mesh VPN — branch independence achieved
- **GitOps model** deployed: ArgoCD reconciling platform state continuously
- **Complete network documentation** for Kantra site: 34 interfaces, 67 policies, 5 tunnels
- **Knowledge transfer** enabled: new engineers have architecture documentation, not just running config

---

## Operational Lessons

### Lesson 1: Documentation is an architectural artifact
The `university-network-architecture` repo wasn't created to satisfy a documentation requirement. It was created because *running a network you can't fully describe is a liability*. The Cairo-S2S VPN failure was diagnosable because the baseline configuration was documented before the failure occurred.

### Lesson 2: Governance overhead is an investment in future speed
Writing 197 ADRs seems like overhead. But each ADR is a conversation that happened once and is recorded forever. Without ADRs, the same conversation happens repeatedly — every time a new engineer joins, every time a stakeholder questions a decision, every time you need to justify a budget line.

### Lesson 3: The platform team is the last team that should build features
The platform team's job is to build infrastructure that makes other teams faster. Every feature built by the platform team is a feature that team can't maintain indefinitely. Design for handoff, not for heroics.

### Lesson 4: "Phase A is a complete product" is a management superpower
When Phase A is defined as a standalone product with value, you can tell stakeholders "we're done with Phase A" instead of "we're 20% of the way to the platform." The framing changes the organizational conversation entirely.

---

## What I Would Do Differently

1. **Start the network documentation earlier.** The Kantra-site as-is documentation was created in December 2025, well after the network was operational. Earlier documentation would have caught the Cairo-S2S Phase 2 failure sooner.

2. **Define the ADR schema before writing the first ADR.** The Nexus Core ADRs evolved their format over time. A consistent schema from ADR-001 would have made the library more navigable earlier.

3. **Train the operations team on GitOps before deploying it.** ArgoCD is intuitive to engineers who think in GitOps terms. For engineers who learned infrastructure through SSH and manual config, the mental model shift requires deliberate onboarding.

---

*This case study documents a platform architecture engagement where the primary deliverable was not code or configuration — it was a governance system that makes infrastructure decisions trustworthy over time.*
