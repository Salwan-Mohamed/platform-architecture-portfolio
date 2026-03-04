# ADR-002: GitOps as the Single Platform Control Plane

**Status:** Accepted  
**Date:** 2025-11-01  
**Decider:** Salwan Mohamed  
**Domain:** Platform  
**Project:** Nexus Platform Engineering / Sinai University

---

## Context

In a multi-site university environment with a small operations team, the primary failure mode is not technical failure — it's **coordination failure**. Changes are made verbally. Configuration drifts from documentation. Nobody knows what the "current state" actually is because current state lives in the heads of 2-3 engineers.

The question: how do you make infrastructure state observable, auditable, and recoverable without adding operational overhead that a small team can't sustain?

---

## Constraints

- Small team (1-2 platform engineers)
- Multi-site environment (3 campuses + datacenter)
- Mixed maturity of operational staff
- No dedicated CI/CD budget — must use available tooling
- University change management process requires audit trails

---

## Decision

**Git is the single source of truth for all platform state. No configuration change is valid unless it is committed to the appropriate repository. ArgoCD provides continuous reconciliation between Git state and cluster state.**

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|-----------------|
| Manual operations with documentation | Documentation becomes stale; no enforcement mechanism; state drift undetectable |
| Ansible-only automation | Push model; no continuous reconciliation; state can drift between runs |
| Terraform-only | Strong for provisioning; weak for Day 2 operations and application deployment |
| Imperative CI/CD pipelines | Pipeline-as-gatekeeper, not platform-as-truth; state is in pipeline logs, not a system |

---

## Trade-offs

**Given up:**
- Ad-hoc changes for urgent situations (GitOps requires a commit, which takes time)
- Operator flexibility to "just fix it" without documentation

**Gained:**
- Every change has a commit SHA, author, timestamp, and message
- State drift is detectable and alertable (ArgoCD out-of-sync)
- Rollback is `git revert` — no manual state reconstruction
- Onboarding new operators means pointing them at Git, not briefing them for 3 days
- University audit requirements satisfied by default — the audit trail is the commit log

---

## Consequences

**Positive:**
- `su_gitops_project` repo contains all GitOps manifests for the platform
- ArgoCD reconciles cluster state continuously — drift is visible in the ArgoCD UI
- The `platform-operating-system` signal registry validates GitOps sync health as a platform signal
- New capabilities are introduced by opening a PR, not by running a script

**Negative:**
- Emergency changes require discipline: even urgent fixes should go through Git (mitigated by pre-approved fast-track templates)
- Learning curve for operators unfamiliar with GitOps model
- ArgoCD becomes a dependency — if ArgoCD is unhealthy, reconciliation stops (mitigated by drift detection alerts)

---

## Evidence

- [su_gitops_project](https://github.com/Salwan-Mohamed/su_gitops_project) — GitOps manifests repository
- [nexus-platform-engineering ADR-0079](https://github.com/Salwan-Mohamed/nexus-platform-engineering) — "Git as Single Control Plane" Core doctrine
- [gitops-mastery-tutorial](https://github.com/Salwan-Mohamed/gitops-mastery-tutorial) — Engineering knowledge developed around GitOps practices
- Platform observability: GitOps sync health is Observability Axis 6 (Platform) in the 8-axis framework

---

## Operational Lesson

The highest-value moment of GitOps was not the happy path — it was the first time a misconfiguration caused a deployment failure. Because the misconfiguration was in Git, the diagnosis was: (1) read the commit, (2) read the diff, (3) understand what changed. Recovery was `git revert`. Total time: 12 minutes. Without GitOps, the same incident would have required interrogating cluster state, reconstructing the change timeline from memory, and manually patching each affected resource.

---

*Reference Core ADR: nexus-platform-engineering/docs/01-decisions/adr/0079-git-single-control-plane.md*
