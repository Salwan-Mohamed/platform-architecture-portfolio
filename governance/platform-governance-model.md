# Platform Governance Model
## Nexus Platform — Sinai University

---

## Governance Philosophy

Governance is not a process layer on top of the platform. **Governance is the platform.**

A platform without governance is a collection of tools. Tools degrade. Decisions made in tools get forgotten. The platform becomes what was convenient, not what was designed.

The Nexus Platform governance model has three mechanisms:
1. **ADRs** — architectural decisions recorded before implementation begins
2. **Phase gates** — advancement requires explicit evidence, not elapsed time
3. **Authority contracts** — who can decide what, bounded, explicit, traceable

---

## Authority Model

```
GIT (Single Control Plane)
  │
  ├──► BASELINE AUTOMATION ──► Infrastructure + ResourceQuotas + Network Policy
  ├──► RUNTIME AUTOMATION ───► Operations (24h backport SLA for incidents)
  ├──► AI AGENTS ──────────► Advisory only. Cannot execute. Cannot approve.
  └──► HUMAN OWNER ───────► Sole decision authority. Final approval for all gates.
```

### Invariants (Cannot Change Without Constitutional Amendment)

| Invariant | Rule |
|-----------|------|
| No Human Gate | Platform capabilities must not require manual approval to consume |
| No SDLC Bypass | Changes follow full software development lifecycle |
| No Post-Hoc Observability | Observability defined before implementation |
| Ownership = Accountability | Unclear ownership blocks capability |
| Tooling Follows Architecture | Tools don't define architecture; architecture defines tools |
| AI Is Not an Actor | AI recommends; humans decide |

---

## ADR Governance

### What Requires an ADR

| Decision Type | ADR Required? |
|--------------|---------------|
| Choosing a new tool or vendor | YES |
| Changing a network topology | YES |
| Introducing a new security boundary | YES |
| Modifying an existing security policy | YES |
| Adding a new platform capability | YES |
| Updating configuration for existing capability | NO (config change, not architecture) |
| Operational runbook updates | NO (operational, not architectural) |

### ADR Lifecycle

```
Proposed → Review (Human Owner) → Accepted | Rejected
                                          │
                                          ▼
                                    Implementation
                                          │
                                          ▼
                               Superseded (when replaced)
                               or Deprecated (when retired)
```

### ADR Authority Hierarchy for Sinai University

```
Nexus Core ADRs (nexus-platform-engineering)
  │ Defines: WHAT capabilities must exist, governance principles, phase boundaries
  │ Authority: IMMUTABLE without exceptional justification
  │
  ▼
 Implementation ADRs (nexus-platform-product-sinai-university)
  │ Defines: HOW capabilities are realized at Sinai University
  │ Authority: Must reference a Core ADR; Core always wins in conflicts
  │
  ▼
 Portfolio ADRs (this repository)
    Defines: Cross-cutting architecture decisions and patterns
    Authority: Documents portfolio-level thinking; informational for other environments
```

---

## Phase Gate Framework

| Phase | Product | Entry Criteria | Exit Criteria |
|-------|---------|---------------|---------------|
| A | Infrastructure Control | Team exists; Git initialized | Stability signals baseline; Core services in Git; no ad-hoc changes |
| B | Explainable Platform | Phase A stable 30+ days | Observability all 8 axes active; SLOs defined; error budgets tracked |
| C | Organizational Memory | Phase B stable 30+ days | Knowledge base operational; self-service catalog defined |
| D | Governed Self-Service | Phase C stable 30+ days | Developer platform active; golden paths operational |

**Critical Rule: Advancement is optional, not assumed. Phase A is a complete, standalone product.**

---

## Change Control

### Change Categories

| Category | Description | Approval Required |
|----------|-------------|------------------|
| P1 - Critical | Security boundaries, network topology, identity | Human Owner + Architecture review |
| P2 - Significant | New capabilities, platform component upgrades | Human Owner |
| P3 - Standard | Configuration updates, runbook changes | PM approval |
| P4 - Routine | Documentation, dashboards, non-breaking | PR approval |

### Change Flow

```
Propose (PR opened)
  │
  ▼
 Review (by severity-appropriate approver)
  │
  ▼
 Evidence check (links to evidence/config required for P1/P2)
  │
  ▼
 Approve → Merge to main → ArgoCD reconciles
  OR
 Reject (with documented reason)
```

---

## Signal-Based Decision Validation

Every platform decision is validated against signals, not opinions.

| Signal Domain | Examples | ADR Reference Required |
|--------------|----------|------------------------|
| Correctness | Policy enforcement, config drift, contract validation | YES for P1/P2 decisions |
| Value | Adoption rate, effort displacement, time-to-production | YES for capability decisions |
| Stability | Error rate, latency variance, incident frequency | YES for architecture decisions |
| Evolution | Time-to-introduce capability, coupling indicators | YES for governance decisions |

**Rule: Signals without owners are invalid. Thresholds defined post-incident are invalid.**

---

*This governance model is the foundation of the Nexus Platform. Without it, the platform is just infrastructure. With it, the platform is an organizational coordination system.*
