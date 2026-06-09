# Salwan Mohamed — Platform Architecture Portfolio

> **Principal Platform Engineer** | Multi-Site Infrastructure Architect | GitOps-First | Zero-Trust Identity | Governed Automation

[![ADRs](https://img.shields.io/badge/ADRs-21-green.svg)](#adr-library)
[![Phase 1](https://img.shields.io/badge/Sinai_Phase_1-CLOSED-brightgreen.svg)](https://github.com/Salwan-Mohamed/nexus-platform-product-sinai-university)
[![Case Studies](https://img.shields.io/badge/Case_Studies-4-orange.svg)](#case-studies)
[![Track](https://img.shields.io/badge/Track-Senior_→_Principal-gold.svg)](#career-track)

---

## Who I Am

I am a Platform Engineer who builds structured infrastructure platforms for multi-site organizations. I don't just operate infrastructure — I design the governance systems, decision frameworks, and organizational coordination layers that make infrastructure trustworthy at scale.

My work spans three interlocking domains:

**Architecture** — Systems with explicit authority models, vendor-agnostic governance, and phase-based maturity that let organizations grow without re-architecting.

**Engineering Storytelling** — Every system I build is documented from first principles: why this, not that. What was the constraint. What was traded. What was learned.

**Organizational Coordination** — Infrastructure is a people problem. My platforms are organizational coordination systems implemented with GitOps — defined by what they coordinate, not what they're built on.

---

## The Nexus Platform System

```
┌───────────────────────────────────────────────────────────────────┐
│                     NEXUS PLATFORM SYSTEM                        │
├───────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Platform Operating System           Signal-driven governance     │
│  ├── Authority contracts             Reality-anchored decisions   │
│  ├── Decision gates                  No implicit approvals        │
│  └── Capability Maturity Model       7-level write-capability     │
│                            │         gate (ADR-017)               │
│                            ▼                                      │
│  Nexus Platform Core (183 ADRs)      Vendor-agnostic doctrine     │
│  ├── Phase A — Infrastructure Control   CLOSED ✓                  │
│  ├── Phase B — Explainable Platform     CLOSED ✓                  │
│  ├── Phase C — Platform Knowledge       ACTIVE                    │
│  └── Phase D — Governed Self-Service    Future                    │
│                            │                                      │
│                            ▼                                      │
│  Sinai University Product            Real-world implementation    │
│  ├── Phase 1 (Nexus Core Platform)   CLOSED ✓ (PR #97, 2026-05)  │
│  │   ├── RKE2 + Cilium + Hubble      Kubernetes + eBPF network   │
│  │   ├── ArgoCD (18 apps synced)     GitOps control plane        │
│  │   ├── Keycloak + AD (1,255 users) SSO across 5 consumers      │
│  │   ├── Prometheus/Grafana/Loki     Full observability stack     │
│  │   ├── Longhorn → MinIO S3         Persistent storage + backup  │
│  │   └── FortiGate 1101E (3 sites)   Full-mesh IPsec VPN         │
│  └── Phase 2                         Portability + hardening      │
│                            │                                      │
│                            ▼                                      │
│  Governance Architecture             5 structural ADRs            │
│  ├── Capability Maturity Model       Write-gate by evidence       │
│  ├── IA Positioning                  Core never depends on IA     │
│  ├── Domain Application Framework    Plugin-contract model        │
│  ├── Integration Rule                4-layer authority hierarchy  │
│  └── Refactory-First Change Model    Complexity by justification  │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

---

## Sinai University Platform — Phase 1 Closed

> **Source:** [nexus-platform-product-sinai-university](https://github.com/Salwan-Mohamed/nexus-platform-product-sinai-university) · PR #97 · HEAD `e7f1d25` · Closed 2026-05-31

Phase 1 (Nexus Core Platform) and Wave 1.7 (Access Foundation) are **CLOSED** — all 14 roadmap criteria satisfied or dispositioned. This is a **foundation reference implementation** for a 3-site, ~5,800-user university: not yet portable or commercially packaged, but fully operational and governance-complete.

### What Was Built and Is Running

| Layer | Component | Version | State |
|-------|-----------|---------|-------|
| Kubernetes | RKE2 | v1.34.x | Operational |
| CNI | Cilium | 1.15.6 | Operational |
| Network observability | Hubble | bundled | Operational |
| GitOps | ArgoCD | v2.13.3 | 18 apps — all Synced + Healthy |
| Storage | Longhorn → MinIO S3 | v1.6.2 | Operational + backup proven |
| Metrics | Prometheus (kube-prometheus-stack) | v2.53.1 | Operational |
| Logging | Loki + Promtail | v2.9.10 | Operational |
| Visualization | Grafana | v11.2.2 | Operational |
| Alerting | Alertmanager | bundled | Operational |
| Identity broker | Keycloak | 26.3.3 | 1,255 users federated from AD |
| Identity source | Active Directory (SU.intra) | — | LDAPS federated |
| SSO | oauth2-proxy + Keycloak OIDC | — | 5 consumers live |

### SSO Coverage — 5 Consumers Live

| Consumer | Class | Mechanism |
|----------|-------|-----------|
| ArgoCD | A — native OIDC | Keycloak OIDC client |
| Grafana | A — native OIDC | Keycloak OIDC client + group→role |
| Longhorn UI | C — no native OIDC | oauth2-proxy + Keycloak OIDC + group gate |
| Alertmanager UI | C — no native OIDC | oauth2-proxy + Keycloak OIDC + group gate |
| Hubble UI | C — no native OIDC | oauth2-proxy + Keycloak OIDC + group gate |

### Honest Boundaries (What Phase 1 Is Not)
- Not portable / multi-customer
- Not commercially packaged or sale-ready
- Production destructive restore: positioned, not yet proven (isolated drill validated)
- Cross-region DR: deferred to Phase 2 (DI-027)
- C5 firewall automation: explicitly firewalled

### Governance Models Produced (Wave 1.7)

| Model | File |
|-------|------|
| SSO 5-class model | `docs/governance/V3.0.1-SSO-MODEL.md` |
| DNS/hostname/redirect-URI model | `docs/governance/V3.0.1-DNS-EXPOSURE-MODEL.md` |
| CA trust-distribution model | `docs/governance/V3.0.1-CA-TRUST-DISTRIBUTION-MODEL.md` |
| On-call / alert-routing model | `docs/governance/V3.0.1-ON-CALL-ROUTING-MODEL.md` |
| Slice Execution Template | `docs/governance/SLICE-EXECUTION-TEMPLATE.md` |

---

## Architecture & Design

### Platform Architecture: Nexus Platform v2.0 Baseline

The authoritative architectural reference for the Sinai implementation is the v2.0 Baseline Candidate — a 918-line document covering the full 9-layer topology, Platform Capability Contracts (PCC), authority model, and completion verification:

> [`docs/architecture/NEXUS-PLATFORM-ARCHITECTURE-V2.0-BASELINE-CANDIDATE.md`](https://github.com/Salwan-Mohamed/nexus-platform-product-sinai-university/blob/main/docs/architecture/NEXUS-PLATFORM-ARCHITECTURE-V2.0-BASELINE-CANDIDATE.md)

The v2.0 architecture was authored in a 6-stage chain (PR #14–#21), each stage merged with evidence verification. Key architectural decisions resolved during authoring:

| Stage | Decision |
|-------|----------|
| Stage 1 | Document foundation — governance protocols, authority hierarchy |
| Stage 2 | Chain A framework — Capability Maturity Model + IA Positioning |
| Stage 3 | Chain B framework — Domain Plugin Model + Integration Rule |
| Stage 4 | Layer topology resolved (Option β — 9-layer model with IA as extension, not core) |
| Stage 5 | Platform Capability Contracts operational scope defined |
| Stage 6 | Completion verification — all forward-pointers resolved |

### Platform Architecture Layers

```
Layer 9 — Governed Self-Service         (Phase D — future)
Layer 8 — Platform Knowledge            (Phase C — active)
Layer 7 — Infrastructure Automation     (Extension — not Core dependency)
Layer 6 — Observability                 Prometheus / Grafana / Loki / Hubble
Layer 5 — Application Platform          ArgoCD / Tekton / Domain contracts
Layer 4 — Identity & Access             Keycloak / AD / OIDC / oauth2-proxy
Layer 3 — Kubernetes                    RKE2 / Cilium / CAPV / Sealed Secrets
Layer 2 — Virtualization                VMware vSphere
Layer 1 — Physical Infrastructure       FortiGate / Aruba / 3-site VPN
Layer 0 — Governance                    183 Core ADRs + 21 Portfolio ADRs
```

### Governance Architecture (ARCH Series)

Five structural ADRs define the operating model that governs how the platform changes and grows:

| ADR | Principle | What It Governs |
|-----|-----------|----------------|
| [ADR-017](architecture-decisions/ADR-017-capability-maturity-model.md) | Write-capability is gated to evidence, not calendar | 7-level maturity progression; no automated write without Human Owner approval |
| [ADR-018](architecture-decisions/ADR-018-infrastructure-automation-positioning.md) | Core platform must never depend on infrastructure automation | Named failure-mode commitments per component; visibility-first per domain |
| [ADR-019](architecture-decisions/ADR-019-domain-application-framework.md) | Applications join via capability contracts, not direct coupling | Domain plugin pattern; platform enforces contracts without domain knowing internals |
| [ADR-020](architecture-decisions/ADR-020-integration-rule.md) | Integration direction follows authority hierarchy | 4-layer model; forbidden couplings enforced at PR time |
| [ADR-021](architecture-decisions/ADR-021-refactory-first-platform-change-model.md) | Addition requires proof of insufficiency | Complexity grows by justification only; deferred components reviewed at phase gates |

---

## ADR Library — 21 Architecture Decision Records

### Platform Governance
| ADR | Decision |
|-----|----------|
| [ADR-001](architecture-decisions/ADR-001-nexus-core-product-separation.md) | Core/Product Separation — vendor-agnostic doctrine vs. environment-specific implementation |
| [ADR-002](architecture-decisions/ADR-002-gitops-as-control-plane.md) | GitOps as the only change mechanism — Git is truth, drift is an incident |
| [ADR-005](architecture-decisions/ADR-005-phase-based-platform-maturity.md) | Phase-based maturity — advancement optional, not assumed |
| [ADR-006](architecture-decisions/ADR-006-signal-driven-decision-governance.md) | Signal-driven decisions — no decision without observable evidence |
| [ADR-008](architecture-decisions/ADR-008-human-in-loop-dc-migration.md) | Human-in-loop for irreversible operations — AI advises, never approves |
| [ADR-012](architecture-decisions/ADR-012-evidence-gated-phase-transitions.md) | Evidence-gated phase transitions — gates, not calendars |
| [ADR-017](architecture-decisions/ADR-017-capability-maturity-model.md) | Capability Maturity Model — 7-level write-capability gate |
| [ADR-018](architecture-decisions/ADR-018-infrastructure-automation-positioning.md) | Infrastructure Automation Positioning — Core/IA independence |
| [ADR-019](architecture-decisions/ADR-019-domain-application-framework.md) | Domain Application Framework — contract-based plugin model |
| [ADR-020](architecture-decisions/ADR-020-integration-rule.md) | Integration Rule — 4-layer authority hierarchy |
| [ADR-021](architecture-decisions/ADR-021-refactory-first-platform-change-model.md) | Refactory-First — complexity by justification only |

### Infrastructure & Network
| ADR | Decision |
|-----|----------|
| [ADR-003](architecture-decisions/ADR-003-rke2-over-vanilla-kubernetes.md) | RKE2 over vanilla Kubernetes — CIS hardening by default, not by effort |
| [ADR-004](architecture-decisions/ADR-004-fortigate-network-boundary.md) | FortiGate as network boundary — team operability over feature maximalism |
| [ADR-007](architecture-decisions/ADR-007-full-mesh-vpn-over-hub-spoke.md) | Full-mesh VPN — no hub SPOF; validated by real 4-hour outage |
| [ADR-010](architecture-decisions/ADR-010-cilium-cni-over-canal.md) | Cilium CNI over Canal default — eBPF, L7 policy, Hubble observability |
| [ADR-014](architecture-decisions/ADR-014-vsphere-capv-cluster-lifecycle.md) | vSphere + CAPV — cluster lifecycle as Git operations |
| [ADR-016](architecture-decisions/ADR-016-infrastructure-execution-stack.md) | Execution stack — Terraform + AWX + Tekton + Sealed Secrets |

### Identity & Observability
| ADR | Decision |
|-----|----------|
| [ADR-009](architecture-decisions/ADR-009-keycloak-identity-broker.md) | Keycloak as identity broker — AD → OIDC with no shadow stores |
| [ADR-011](architecture-decisions/ADR-011-slo-driven-alerting.md) | SLO-driven alerting — burn rate signals, not raw metric thresholds |
| [ADR-013](architecture-decisions/ADR-013-argocd-hub-spoke-gitops.md) | ArgoCD hub-spoke — single reconciliation authority across all clusters |
| [ADR-015](architecture-decisions/ADR-015-four-component-observability-stack.md) | Four-component observability — Prometheus + Grafana + Loki + Robusta |

---

## Case Studies

| Study | Summary | Complexity |
|-------|---------|------------|
| [CS-01](case-studies/CS-01-sinai-university-platform.md) | Nexus Platform for Sinai University — Phase 1 closed, 21 ADRs, SSO live | ⭐⭐⭐⭐⭐ |
| [CS-02](case-studies/CS-02-platform-operating-system/README.md) | Signal-Driven Platform Operating System | ⭐⭐⭐⭐⭐ |
| [CS-03](case-studies/CS-03-university-network-architecture/README.md) | Multi-Site Network Architecture — 3-campus full-mesh VPN | ⭐⭐⭐⭐ |
| [CS-04](case-studies/CS-04-dc-migration-governance/README.md) | Governance-First Datacenter Migration | ⭐⭐⭐⭐ |

---

## System Diagrams

| Diagram | Description |
|---------|-------------|
| [Nexus Platform Layers](system-diagrams/nexus-platform-layers.md) | Full 9-layer platform stack |
| [University Network Topology](system-diagrams/university-network-topology.md) | 3-site full-mesh IPsec with VPN status |
| [Governance Authority Model](system-diagrams/governance-authority-model.md) | Decision authority by layer |
| [LinkedIn Architecture Posts](linkedin/posts/) | 3 posts with embedded Mermaid diagrams |

---

## Technology Fingerprint

| Domain | Technologies |
|--------|--------------|
| **Kubernetes** | RKE2, Cluster API (CAPV), Cilium 1.15.6, Hubble |
| **GitOps** | ArgoCD v2.13.3, Terraform, Ansible/AWX, Tekton |
| **Identity** | Keycloak 26.3.3, Active Directory, OIDC, oauth2-proxy |
| **Security** | FortiGate 1101E, Sealed Secrets, Zero-Trust boundary, CA/cert-manager |
| **Virtualization** | VMware vSphere, CAPV |
| **Observability** | Prometheus v2.53.1, Grafana v11.2.2, Loki v2.9.10, Robusta, Hubble |
| **Storage** | Longhorn v1.6.2, MinIO S3 backup |
| **Network** | FortiGate, Aruba, full-mesh IPsec VPN, 3-site BGP-routed |
| **Governance** | 21 Portfolio ADRs, 5 ARCH governance ADRs, Phase-gate model |

---

## Architecture Thinking Signals

> These are the questions I ask before writing a line of config:

1. **Who has authority here?** — Authority must be explicit, documented, and bounded.
2. **What signals validate this decision?** — Opinions don't override data.
3. **What phase is this organisation in?** — Advancement is optional, not assumed.
4. **What is the blast radius?** — Every change has a containment model.
5. **What happens when this fails?** — Rollback is a first-class design concern.
6. **What maturity level is this capability at?** — Write-capability requires evidence, not just deployment.
7. **What am I NOT doing?** — Good architecture is as much about constraints as capabilities.

---

*"Nexus is defined by what it coordinates, not what it's built on."*

**Salwan Mohamed** | Platform Engineer | [GitHub](https://github.com/Salwan-Mohamed) | [Sinai University Platform](https://github.com/Salwan-Mohamed/nexus-platform-product-sinai-university)
