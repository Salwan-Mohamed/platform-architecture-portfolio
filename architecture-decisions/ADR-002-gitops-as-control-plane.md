# ADR-002: GitOps as the Single Control Plane

**Status:** Accepted  
**Date:** 2025-12-07  
**Decider:** Salwan Mohamed  
**Project:** Nexus Platform Engineering + Sinai University  

---

## Context

Before the Nexus platform, infrastructure changes at Sinai University were applied manually via CLI, SSH into switches, direct vSphere console modifications, and "tribal knowledge" runbooks stored in personal OneDrive folders. There was no audit trail, no change review, no rollback mechanism, and no way to answer the question: **"who changed this, when, and why?"**

During one incident, a network change applied directly to the core switch caused a 40-minute outage across two sites. No one could identify who made the change or why. Recovery required guessing at the previous state from memory.

The platform needed a single source of truth for all infrastructure state — one that was auditable, reviewable, and recoverable.

---

## Constraints

| Constraint | Impact |
|------------|--------|
| University IT governance requires change audit trails | All changes must be traceable to a named human with a documented reason |
| Mixed-skill ops team (senior + junior) | Cannot require CLI expertise for all changes; needs approachable interface |
| Limited rollback tooling in current state | Recovery from bad changes is manual and slow |
| 3 sites with partially overlapping responsibility | Without coordination layer, changes in one site can conflict with another |
| FortiGate, vSphere, RKE2 — three different control planes | Need unified operational model across dissimilar systems |

---

## Decision

**Git is the single control plane for all infrastructure state. No configuration change is valid if it is not expressed in Git. Automation (ArgoCD, Terraform, Ansible/AWX) reads desired state from Git and reconciles it to reality.**

The operational model:
- Desired state lives in Git (the repository is truth)
- ArgoCD continuously reconciles Kubernetes state from Git
- Terraform manages vSphere and network infrastructure state from Git-committed plans
- AWX executes runbooks triggered by Git-managed Ansible playbooks
- Direct CLI changes are treated as drift and flagged for remediation

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|----------------|
| **ITSM ticket system as source of truth** | Tickets describe intent; they don't represent actual state or enable automated reconciliation |
| **Configuration Management Database (CMDB)** | CMDBs describe state at a point in time; they drift from reality and require manual synchronization |
| **Terraform Cloud/Atlantis only** | Covers infrastructure provisioning but not application deployment or operational runbooks |
| **Manual documented runbooks** | No automated enforcement; relies on operator discipline; known failure mode from incident history |

---

## Trade-offs

**What we gained:**
- Every change has a commit, an author, a PR review, and a reason (commit message / ADR reference)
- Rollback is `git revert` + ArgoCD sync — tested and reliable
- Drift detection is automated: ArgoCD/Terraform alerts when actual state diverges from Git state
- Junior operators can make changes via PR review process with senior oversight built in
- University auditors can inspect the full change history without needing access to production systems

**What we gave up:**
- Speed of "just SSH in and fix it" — emergency changes now require a PR (with an expedited path)
- Initial cultural resistance from ops team who preferred direct CLI access
- Bootstrap complexity: chicken-and-egg problem when Git infrastructure itself needs to be provisioned

---

## Consequences

- ArgoCD deployed as the Kubernetes GitOps reconciler for all cluster state
- `su_gitops_project` repository implements the full GitOps pattern for Sinai University
- All Aruba switch configurations are now managed via Ansible playbooks in Git (`network-automation-aruba`)
- Drift detection alerts are configured in Grafana dashboard linked to ArgoCD sync status
- The incident pattern of "unknown who changed what" has been eliminated in covered infrastructure

---

## Evidence

- [su_gitops_project](https://github.com/Salwan-Mohamed/su_gitops_project) — Sinai University GitOps implementation
- [network-automation-aruba](https://github.com/Salwan-Mohamed/network-automation-aruba) — Network state in Git
- [gitops-mastery-tutorial](https://github.com/Salwan-Mohamed/gitops-mastery-tutorial) — GitOps patterns documentation
- Core ADR-0079: Git as Single Control Plane
- Core ADR-0142: Intent Contracts Before Automation

---

## Principal-Level Signal

> GitOps is not about using ArgoCD. It is a governance decision: **you are committing to the principle that the repository is truth, and reality must conform to it — not the other way around.** Teams that adopt the tool without the governance principle end up with ArgoCD configured to allow manual overrides, which is the worst of both worlds: overhead of GitOps, none of the auditability.
