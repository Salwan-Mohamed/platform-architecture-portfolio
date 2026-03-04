# Salwan Mohamed — Platform Architecture Portfolio

> **Principal Platform Engineer** | Multi-Site Infrastructure Architect | GitOps-First | Zero-Trust Security | Organizational Platform Coordination

[![Nexus Core](https://img.shields.io/badge/Nexus_Core-v14.0-blue.svg)](https://github.com/Salwan-Mohamed/nexus-platform-engineering)
[![ADRs](https://img.shields.io/badge/ADRs-183-green.svg)](#adr-library)
[![Case Studies](https://img.shields.io/badge/Case_Studies-4-orange.svg)](#case-studies)
[![Track](https://img.shields.io/badge/Track-Senior_→_Principal-gold.svg)](#career-track)

---

## Who I Am

I am a Platform Engineer who builds structured infrastructure platforms for multi-site organizations. I don't just operate infrastructure — I design the governance systems, decision frameworks, and organizational coordination layers that make infrastructure trustworthy at scale.

My work spans three interlocking domains:

**Architecture** — I design systems with explicit authority models, vendor-agnostic governance, and phase-based maturity that let organizations grow without re-architecting.

**Engineering Storytelling** — Every system I build is documented from first principles: why this, not that. What was the constraint. What was traded. What was learned.

**Organizational Coordination** — Infrastructure is a people problem. My platforms are organizational coordination systems implemented with GitOps — defined by what they coordinate, not what they're built on.

---

## The Nexus Platform System

The work across my career converges into a coherent platform system:

```
┌──────────────────────────────────────────────────────────────────┐
│                    NEXUS PLATFORM SYSTEM                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Platform Operating System          Signal-driven governance     │
│  ├── Authority contracts            Reality-anchored decisions   │
│  ├── Decision gates                 No implicit approvals        │
│  └── Fitness functions              Enforcement > convenience    │
│                           │                                      │
│                           ▼                                      │
│  Nexus Platform Core (183 ADRs)     Vendor-agnostic doctrine     │
│  ├── Phase A: Infrastructure Control                             │
│  ├── Phase B: Explainable Platform                               │
│  ├── Phase C: Organizational Memory                              │
│  └── Phase D: Governed Self-Service                              │
│                           │                                      │
│                           ▼                                      │
│  Sinai University Product           Real-world implementation    │
│  ├── FortiGate 1101E (14 ADRs)      3 sites, full-mesh VPN      │
│  ├── RKE2 + Cilium + ArgoCD         GitOps-first Kubernetes     │
│  ├── VMware vSphere                 Virtualization layer        │
│  └── Keycloak + AD Identity         Zero-trust boundary         │
│                           │                                      │
│                           ▼                                      │
│  DC Migration Platform              Governance-first migration   │
│  ├── Phase-gated execution          Human-in-loop always        │
│  ├── Evidence-based approvals       Nothing agreed without docs  │
│  └── Risk traceability              Every decision has a owner   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Portfolio Contents

### 📐 Architecture Decisions (ADR Library)
| ADR | Title | Domain |
|-----|-------|--------|
| [ADR-001](architecture-decisions/ADR-001-nexus-core-product-separation.md) | Core/Product Separation Model | Governance |
| [ADR-002](architecture-decisions/ADR-002-gitops-as-control-plane.md) | GitOps as Control Plane | Platform |
| [ADR-003](architecture-decisions/ADR-003-rke2-over-vanilla-kubernetes.md) | RKE2 over Vanilla Kubernetes | Infrastructure |
| [ADR-004](architecture-decisions/ADR-004-fortigate-network-boundary.md) | FortiGate as Network Boundary | Security |
| [ADR-005](architecture-decisions/ADR-005-phase-based-platform-maturity.md) | Phase-Based Platform Maturity Model | Strategy |
| [ADR-006](architecture-decisions/ADR-006-signal-driven-decision-governance.md) | Signal-Driven Decision Governance | Operations |
| [ADR-007](architecture-decisions/ADR-007-full-mesh-vpn-over-hub-spoke.md) | Full-Mesh VPN over Hub-Spoke | Network |
| [ADR-008](architecture-decisions/ADR-008-human-in-loop-dc-migration.md) | Human-in-Loop for DC Migration | Governance |

### 🔬 Case Studies
| Study | Summary | Complexity |
|-------|---------|------------|
| [CS-01](case-studies/CS-01-nexus-platform-university/README.md) | Building a Platform Architecture for Sinai University | ⭐⭐⭐⭐⭐ |
| [CS-02](case-studies/CS-02-platform-operating-system/README.md) | Designing a Signal-Driven Platform Operating System | ⭐⭐⭐⭐⭐ |
| [CS-03](case-studies/CS-03-university-network-architecture/README.md) | Multi-Site Network Architecture for a University | ⭐⭐⭐⭐ |
| [CS-04](case-studies/CS-04-dc-migration-governance/README.md) | Governance-First Datacenter Migration Platform | ⭐⭐⭐⭐ |

### 🏗️ System Diagrams
| Diagram | Description |
|---------|-------------|
| [Nexus Platform Layers](system-diagrams/nexus-platform-layers.md) | Full platform stack visualization |
| [University Network Topology](system-diagrams/university-network-topology.md) | 3-site full-mesh IPsec architecture |
| [Governance Authority Model](system-diagrams/governance-authority-model.md) | Who can decide what, at each layer |
| [Platform Phase Roadmap](system-diagrams/platform-phase-roadmap.md) | A→B→C→D maturity model |

### 🏛️ Governance Models
| Document | Description |
|----------|-------------|
| [Platform Governance Model](governance/platform-governance-model.md) | Core governance principles and enforcement |
| [Multi-Site Operations](governance/multi-site-operations.md) | Operating across 3 campus sites |
| [Change Control Framework](governance/change-control-framework.md) | How changes are approved and traced |

### 💼 Career Track
| Document | Description |
|----------|-------------|
| [Career Narrative](CAREER-NARRATIVE.md) | Senior → Principal trajectory story |
| [LinkedIn Profile](linkedin/README.md) | Ready-to-paste LinkedIn content |
| [Technical Identity](linkedin/technical-identity.md) | Headline, About, and positioning |

---

## Technology Fingerprint

| Domain | Technologies |
|--------|--------------|
| **Kubernetes** | RKE2, Cluster API (CAPV), Cilium, Tekton |
| **GitOps** | ArgoCD, Flux CD, Terraform, Ansible/AWX |
| **Security** | FortiGate 1101E, Keycloak, Active Directory, Zero-Trust |
| **Virtualization** | VMware vSphere |
| **Observability** | Prometheus, Grafana, Loki (8-axis framework) |
| **Network** | Aruba switches, IPsec VPN, full-mesh topology |
| **Governance** | ADR-driven, phase-gated, signal-validated |
| **Automation** | Ansible, AWX, Python, Bash |

---

## Architecture Thinking Signals

> These are the questions I ask before writing a line of config:

1. **Who has authority here?** — Authority must be explicit, documented, and bounded.
2. **What signals validate this decision?** — Opinions don't override data.
3. **What phase is this organization in?** — Advancement is optional, not assumed.
4. **What is the blast radius?** — Every change has a containment model.
5. **What happens when this fails?** — Rollback is a first-class design concern.
6. **Who owns this 2 years from now?** — Unclear ownership blocks capability.
7. **What am I NOT doing?** — Good architecture is as much about constraints as capabilities.

---

*"Nexus is defined by what it coordinates, not what it's built on."*

**Salwan Mohamed** | Platform Engineer | [GitHub](https://github.com/Salwan-Mohamed)
