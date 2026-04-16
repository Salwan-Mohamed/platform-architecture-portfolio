# Case Study CS-01: Nexus Platform for Sinai University

**Status:** Updated  
**Last updated:** 2026-04-16 (sync with IMPL-0021–0031)  
**Complexity:** ⭐⭐⭐⭐⭐  
**Duration:** Phase A complete; Phase B active

---

## The Problem

3 campuses. 5,800 users. No audit trail for network changes. A 40-minute unexplained outage with no post-mortem because no one could answer: who changed this, and what was the state before?

The university needed a platform that a governance board could audit, a small ops team could operate at 2am, and a third campus could join without rebuilding the architecture.

---

## Environment

| Dimension | Detail |
|-----------|--------|
| Sites | 3 campuses: Kantra (main), Katamia, Arish |
| Users | ~5,800 staff, faculty, students |
| Identity | Active Directory (`su.intra`), DCs: KANSYSDC01/02 |
| Compute | VMware vSphere |
| Network | FortiGate-1101E (Kantra), Cisco ASA 5545-X (Katamia), full-mesh IPsec |
| Kubernetes | RKE2 with CIS hardening |
| Team | 1 principal + 3 site ops |
| Budget | Mid-enterprise university tier; no cloud spend |

---

## Architecture Overview

The platform is built in two layers: the Nexus Core (183 vendor-agnostic ADRs defining capability contracts) and the Sinai University Product (the implementation of those contracts in specific tools).

Six layers compose the platform stack:

1. **Governance** — 183 Core ADRs + 11 Product IMPLs; phase gate model; signal-driven decisions
2. **Network** — FortiGate full-mesh IPsec VPN; 6 trust zones per site; FortiManager centralized policy
3. **Virtualization** — VMware vSphere; RKE2 on VMs; CIS hardening profile
4. **Kubernetes** — RKE2; Cilium CNI (eBPF); Hubble network observability; Nginx Ingress
5. **Identity** — Active Directory → Keycloak (OIDC broker) → all platform services via OIDC
6. **Observability** — Prometheus + Grafana + Loki + Alertmanager; SLO-driven alerting; Hubble network flows

---

## Key Decisions Made

### Core/Product Separation (ADR-001)
The Nexus Core defines what the platform must do (183 ADRs). The Sinai Product defines how it does it (11 IMPL ADRs). When hardware changes or a vendor is replaced, the Core doesn't move. This separation is the reason a third campus can be added without rebuilding the architecture.

### GitOps as the Only Change Mechanism (ADR-002)
Git is the source of truth. ArgoCD reconciles Kubernetes state. AWX executes Ansible for switch and firewall configuration. No manual CLI changes are permitted. The 40-minute outage with no post-mortem was the last unauthorized change — because there was no Git commit to trace. After GitOps adoption: zero unauthorized changes.

### RKE2 over Vanilla Kubernetes (ADR-003)
RKE2 provides CIS-hardened Kubernetes by default on vSphere. The alternative (kubeadm) required manual CIS hardening work for a feature RKE2 delivers out of the box. For a 4-person team managing 3 sites, that operational reduction matters.

### Cilium CNI over Canal Default (ADR-010) — *Added Phase B*
RKE2's default Canal CNI was explicitly superseded. Cilium (eBPF-based) provides L7-aware network policies, native Hubble observability, and Gateway API readiness. Canal provided none of these. Hubble is deployed alongside Cilium — every network flow between pods is observable, every dropped packet has a visible reason.

### Keycloak as Identity Broker (ADR-009) — *Added Phase B*
Active Directory is the identity source of truth. Keycloak sits between AD and all platform services — federating via LDAPS, issuing OIDC tokens, routing AD group membership into platform RBAC claims. No shadow identity stores. No local passwords. Every service authentication event is auditable through Keycloak. SSO works across ArgoCD, Grafana, and Alertmanager from the first service deployed.

### Full-Mesh VPN over Hub-and-Spoke (ADR-007)
Full-mesh IPsec with BGP between all 3 sites (3 tunnels: Kantra↔Arish, Kantra↔Katamia, Arish↔Katamia). When Kantra-site had a 4-hour ISP outage, Katamia and Arish continued operating normally — AD replication held, no escalation. Hub-and-spoke would have broken inter-branch connectivity for the duration.

### SLO-Driven Alerting (ADR-011) — *Added Phase B*
Alerts fire on SLO burn rate signals, not raw metric thresholds. SLOs are defined per service before any alerting rule is written. CPU > 80% does not page — but "Keycloak is consuming its error budget at 6x sustainable rate" does. For a small team, alert quality is not optional.

### Evidence-Gated Phase Transitions (ADR-012) — *Added Phase B*
Phase advancement requires documented evidence satisfying gate criteria. Phase A gate: 100% of infrastructure changes in Git, zero unauthorized changes. Gate met — Phase A declared complete. Phase B gate: SLOs defined for all Tier-1 services with 30-day baseline. Gate in progress.

---

## Outcomes

| Metric | Before | After |
|--------|--------|-------|
| Infrastructure change auditability | 0% (no trail) | 100% (Git commit history) |
| Unauthorized changes | Unknown frequency | Zero since GitOps adoption |
| VPN inter-site resilience | Hub-dependent (planned) | Full-mesh, hub-independent |
| Authentication model | Service-by-service (none deployed) | SSO via Keycloak OIDC |
| Network observability | None | Per-flow visibility via Hubble |
| Alert quality | No alerting | SLO burn rate, high signal |
| Multi-site outage resilience | Untested | Validated: Kantra 4hr outage, branches unaffected |

---

## What Made It Hard

**Identity is the hardest layer.** Not technically — operationally. Getting AD group structure to map cleanly to platform RBAC roles required understanding the university's organizational structure, which was partially undocumented. The Keycloak federation design took three iterations before the group-to-role chain was correct.

**Cross-vendor VPN is a real operational constraint.** The Kantra↔Katamia tunnel runs IKEv1 Aggressive Mode with SHA-1 because Cisco ASA 5545-X is the Katamia edge device. FortiGate prefers IKEv2. The tunnel works — 10 of 54 proxy IDs are active — but 44 selectors remain inactive due to negotiation failures. This is documented, not resolved. The resolution requires ASA replacement or migration.

**Governance theater is a constant risk.** When Claude Code generates ADRs, there is a risk the process becomes: Claude writes ADR → Human approves → ADR is filed → nothing changes. IMPL-0027 (evidence-gated phase transitions) is the structural response: advancement requires evidence, not approval. The ADR must describe a decision that actually governs the infrastructure.

---

## What I Would Do Differently

**Start Keycloak design in Phase A.** Identity architecture took longer than expected in Phase B because AD OU structure was undocumented. A Phase A identity audit (who are the groups, what do they represent, what does platform RBAC need) would have made the Phase B Keycloak implementation faster.

**Formally decide CNI earlier.** Canal was the implicit default until Phase B. IMPL-0029 is an amendment to the Phase B plan — it would have been cleaner as a Phase A decision. Any time a default is accepted without a formal decision, there is a risk the default is wrong for the actual requirements.

**Document the boring decisions too.** IMPL-0028 (infrastructure execution parameters — node sizing, resource limits) is the least exciting ADR in the library. It is also the one that prevents the most arguments. "What size VMs do we run RKE2 on?" is a question that gets answered many times without this document.

---

## Portfolio Signals

- Core/Product separation scales across sites without rebuilding governance
- GitOps is a governance decision — the tooling is secondary
- Identity is not a late-phase concern — it determines every service's authentication model from day one
- Cilium CNI is a 5-year infrastructure decision, not a default to accept passively
- Phase gates enforce the difference between described capability and demonstrated capability
- Full-mesh VPN resilience is only visible when a hub fails — design for the failure, not the steady state

---

## Source References

| ADR | Topic |
|-----|-------|
| ADR-001 | Core/Product separation |
| ADR-002 | GitOps as control plane |
| ADR-003 | RKE2 over vanilla Kubernetes |
| ADR-004 | FortiGate as network boundary |
| ADR-005 | Phase-based platform maturity |
| ADR-007 | Full-mesh VPN over hub-and-spoke |
| ADR-009 | Keycloak as identity broker |
| ADR-010 | Cilium CNI over Canal |
| ADR-011 | SLO-driven alerting model |
| ADR-012 | Evidence-gated phase transitions |
| IMPL-0021–0031 | Sinai University implementation ADRs (source repo) |
