# Platform Architecture Portfolio — Master Boot Contract

> This file is read by Claude Code when invoked in this repository.
> It defines the agent system, authority model, and operational rules for maintaining this portfolio.

---

## Repository Identity

**Owner:** Salwan Mohamed  
**GitHub:** https://github.com/Salwan-Mohamed  
**Purpose:** Principal Platform Engineer architecture portfolio  
**Portfolio URL:** https://github.com/Salwan-Mohamed/platform-architecture-portfolio  

---

## Agent System Overview

This repository operates a 4-agent system. Each agent has a bounded role.
Agent boot contracts live in `agents/[agent-name]/CLAUDE.md`.

| Agent | Directory | Activation Phrase | Authority |
|-------|-----------|------------------|-----------|
| Discovery Agent | `agents/discovery-agent/` | `Discovery Agent: analyze [repo]` | READ only |
| Architect Agent | `agents/architect-agent/` | `Architect Agent: generate ADR from [report]` | DRAFT to branch |
| Storyteller Agent | `agents/storyteller-agent/` | `Storyteller Agent: generate case study for [project]` | DRAFT to branch |
| Positioning Agent | `agents/positioning-agent/` | `Positioning Agent: generate LinkedIn content` | DRAFT only |

---

## Authority Rules (Non-Negotiable)

1. **Human Owner is sole decision authority.** Salwan Mohamed approves all content before it merges to `main`.
2. **Agents RECOMMEND — human DECIDES.** No agent auto-merges to `main`.
3. **Evidence-only outputs.** No agent invents capabilities not evidenced in source repositories.
4. **Draft branches only.** All agent content goes to `draft/*` branches, not directly to `main`.
5. **AI is not an actor.** Claude Code executes and generates; it does not approve or finalize.

---

## Source Repositories (Primary)

| Repository | Role | Privacy |
|------------|------|---------|
| `nexus-platform-engineering` | Platform Core — 183 ADRs | Private |
| `nexus-platform-product-sinai-university` | Sinai University Product | Private |
| `platform-operating-system` | Signal-driven governance OS | Private |
| `dc-migration-platform` | DC Migration governance platform | Private |
| `university-network-architecture` | Network topology & security design | Private |
| `su_gitops_project` | GitOps implementation | Public |
| `kubernetes-ha-cluster` | RKE2 HA cluster | Public |
| `network-automation-aruba` | Aruba switch automation | Public |
| `cilium-deep-dive-labs` | Cilium/eBPF deep dive | Public |

---

## Portfolio Content Map

```
platform-architecture-portfolio/
├── README.md                          ← Engineering identity + portfolio index
├── CAREER-NARRATIVE.md                ← Senior → Principal trajectory
├── CLAUDE.md                          ← This file: agent bootstrap contract
│
├── agents/                            ← Claude Code agent system
│   ├── AGENT-SYSTEM.md               ← Agent architecture overview
│   ├── discovery-agent/CLAUDE.md     ← Discovery Agent boot contract
│   ├── architect-agent/CLAUDE.md     ← Architect Agent boot contract
│   ├── storyteller-agent/CLAUDE.md   ← Storyteller Agent boot contract
│   ├── positioning-agent/CLAUDE.md   ← Positioning Agent boot contract
│   └── discovery-reports/            ← Agent output: discovery intelligence
│
├── architecture-decisions/            ← ADR Library (8 ADRs)
│   ├── ADR-001: Core/Product Separation
│   ├── ADR-002: GitOps as Control Plane
│   ├── ADR-003: RKE2 over Vanilla K8s
│   ├── ADR-004: FortiGate Network Boundary
│   ├── ADR-005: Phase-Based Maturity Model
│   ├── ADR-006: Signal-Driven Governance
│   ├── ADR-007: Full-Mesh VPN over Hub-Spoke
│   └── ADR-008: Human-in-Loop DC Migration
│
├── case-studies/                      ← 4 Case Studies
│   ├── CS-01: Nexus Platform for University
│   ├── CS-02: Platform Operating System
│   ├── CS-03: Multi-Site Network Architecture
│   └── CS-04: DC Migration Governance
│
├── system-diagrams/                   ← Mermaid architecture diagrams
│   ├── nexus-platform-layers.md
│   └── governance-authority-model.md
│
├── governance/                        ← Governance models
│   ├── platform-governance-model.md
│   └── multi-site-operations.md
│
└── linkedin/                          ← LinkedIn career content
    └── README.md                      ← Headlines, About, Projects, Posts
```

---

## Running the Full Pipeline

```bash
# Step 1: Discovery (read source repos, extract signals)
Discovery Agent: analyze nexus-platform-product-sinai-university
Discovery Agent: analyze nexus-platform-engineering
Discovery Agent: analyze platform-operating-system
Discovery Agent: analyze dc-migration-platform
Discovery Agent: analyze university-network-architecture

# Step 2: Architecture (generate additional ADRs)
Architect Agent: generate ADR from sinai-university-report
Architect Agent: generate ADR from nexus-core-report

# Step 3: Storytelling (generate additional case studies)
Storyteller Agent: generate case study for Nexus Platform University
Storyteller Agent: generate case study for DC Migration Platform

# Step 4: Positioning (update LinkedIn content)
Positioning Agent: generate LinkedIn content
Positioning Agent: generate post about Core/Product separation
```

---

## Current Portfolio Status

| Section | Status | Count |
|---------|--------|-------|
| ADR Library | ✅ Complete (v1) | 8 ADRs |
| Case Studies | ✅ Complete (v1) | 4 studies |
| System Diagrams | ✅ Complete (v1) | 2 diagram sets |
| Governance Models | ✅ Complete (v1) | 2 models |
| LinkedIn Content | ✅ Complete (v1) | Full package |
| Agent System | ✅ Deployed | 4 agents |

**Next actions:**
- Run Discovery Agent on all 5 priority repos for deeper signal extraction
- Generate ADR-009+ from discovery reports
- Write Medium article based on CS-01 (Nexus Platform University)
- Add `platform-phase-roadmap.md` to system-diagrams

---

*"Nexus is defined by what it coordinates, not what it's built on."*
