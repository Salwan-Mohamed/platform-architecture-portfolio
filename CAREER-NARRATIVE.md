# Career Narrative — Senior → Principal Platform Engineer

## The Arc

My engineering career follows a single thread: **making infrastructure legible**.

Not just running — legible. Governable. Explainable to the next engineer, the next team, the next crisis at 2 AM.

This portfolio documents that journey across three distinct phases.

---

## Phase 1 — Infrastructure Engineer: Making Things Work

**The problem I solved**: Systems were fragile, changes were manual, knowledge lived in heads.

**What I built**:
- Kubernetes HA clusters with Ansible automation (`kubernetes-ha-cluster`, `ansible-role-k8s-ha-cluster`)
- Network automation for Aruba switches via AWX/Ansible — replacing manual CLI work with reproducible playbooks
- Nagios monitoring stacks, Cisco CUCM backup automation, GitOps workflows

**What I learned**: You can automate individual tasks indefinitely. That's not a platform. A platform is what you build when you realize the automation itself needs governance.

---

## Phase 2 — Platform Engineer: Making Systems Governable

**The problem I solved**: Automation existed everywhere but was ungoverned. Who owns this? What does it do? What happens when it breaks?

**What I built**:

### Nexus Platform Engineering Core
A **vendor-agnostic governance foundation** — 183 Architecture Decision Records across 11 domain chapters. Not a tool, not a product: a *decision system* that any organization can implement with their own technology choices.

Key insight: Platform governance is not about tools. It's about making decisions visible, traceable, and defensible.

### Sinai University Platform Product
Applied the Nexus Core to a real production environment: **14 capabilities** mapped to specific vendor choices, each backed by an ADR referencing Core principles. FortiGate for network control. RKE2 for Kubernetes. ArgoCD for GitOps. Keycloak for identity. Prometheus + Grafana + Loki for observability.

Key insight: The power of Core/Product separation — the governance model is reusable; the implementation is disposable.

### University Network Architecture (3 sites)
Designed full-mesh IPsec VPN architecture for Sinai University — eliminating hub dependency between Kantra, Katamia, and Arish branches. FortiGate-1101E at each site, autonomous local internet breakout, documented as-is state for every interface and policy.

**What I learned**: Governance documentation IS engineering work. An undocumented architecture is a liability. I built the discipline of documenting architecture decisions before, during, and after implementation.

---

## Phase 3 — Principal Track: Making Organizations Think in Systems

**The problem I'm solving**: Even well-built platforms drift. Decisions degrade. Context evaporates. The engineers who built it leave.

**What I'm building**:

### Platform Operating System
A **signal-driven decision governance framework** — not a product, a meta-system. It enforces decision quality through explicit authority models, signal registries, and gated decision flows. Decisions without signal references are rejected. No implicit approvals. Human owner as sole decision authority.

### DC Migration Platform
A **governance-first migration program**: single source of truth, phase gates with explicit sign-off, decision severity ratings, evidence requirements. Designed so that *nothing is agreed unless documented, nothing is executed unless approved*.

### Salwan Task OS
A personal thinking infrastructure — Git-backed, file-as-truth, Python CLI. Built because I needed a system that enforces the same decision discipline on personal work that I build into platform systems.

---

## The Principal Differentiator

Most Senior Engineers ask: *"What's the best tool for this?"*

Principal Engineers ask: *"What decision are we making, who has authority to make it, what signals tell us if we're right, and how will the engineer three years from now understand why we did this?"*

Every project in this portfolio is evidence of that shift.

---

## Technical Depth Map

| Capability | Depth | Evidence |
|------------|-------|----------|
| Platform Architecture | Principal | Nexus Core — 183 ADRs, 4-phase model |
| Kubernetes Production | Senior | RKE2 HA, CAPV, Cilium, Tekton |
| Network Architecture | Senior | FortiGate full-mesh, IPsec, VLAN design |
| GitOps | Senior | ArgoCD, Terraform, Ansible at scale |
| Governance Design | Principal | ADR methodology, authority models, phase gates |
| Observability Architecture | Senior | 8-axis framework, Prometheus/Grafana/Loki |
| Identity & Security | Senior | Keycloak, AD, OIDC, zero-trust network |
| Decision Systems | Principal | Platform OS, signal-driven governance |
