# ADR-016: Infrastructure Execution Stack

**Status:** Accepted  
**Date:** 2026-04-17  
**Source:** IMPL-0011 (Terraform), IMPL-0012 (AWX), IMPL-0013 (Tekton), IMPL-0015 (Sealed Secrets), IMPL-0018 (Credential Lifecycle)  
**Project:** Sinai University Platform

---

## Context

A GitOps platform requires execution tooling — the layer that translates Git-committed desired state into actual infrastructure. ArgoCD (ADR-013) handles Kubernetes resource reconciliation. But three other execution domains exist: infrastructure provisioning (VMs, DNS, network objects), OS and application configuration, and CI pipeline execution (build, test, image).

Without explicit decisions here, execution tooling proliferates: different engineers use different tools, credentials are stored in different places, execution authority is fragmented. The result is a platform where GitOps governs Kubernetes state but manual operations govern everything else — defeating the governance objective.

Secrets are the hardest part of GitOps execution. Sealed Secrets and credential lifecycle management resolve the question: how do secrets get into Git safely?

## Constraints

- All execution must be auditable via Git commit history
- Credentials must never be stored in plaintext in Git
- CI must be separate from CD — the build system must not have production cluster credentials
- AWX must be the only interface for running Ansible against infrastructure — no ad-hoc ansible-playbook commands from engineer laptops
- Terraform state must be stored centrally — not on engineer laptops

## Decision

**Three-tool execution stack with strict domain separation: Terraform (infrastructure), AWX + Ansible (configuration), Tekton (CI). Sealed Secrets for secret management in Git. Credential rotation defined as a governance process.**

**Terraform:** Provisions infrastructure objects — vSphere VMs (non-Kubernetes), DNS records, network objects, vSphere resource pools. Terraform state stored in a remote backend (not local). Infrastructure modules committed to Git. `terraform apply` runs in an AWX job or CI pipeline, never from an engineer's laptop.

**AWX + Ansible:** Executes configuration against existing infrastructure — OS hardening, package installation, RKE2 bootstrapping on CAPV-provisioned nodes, application configuration, firewall rule application. AWX is the only interface for running Ansible: engineers submit jobs through the AWX UI or API; ad-hoc Ansible from laptops is prohibited. All playbooks in Git. AWX job templates reference Git playbooks.

**Tekton:** CI pipelines — build container images, run tests, push to registry, update image tag in the GitOps repo. Runs in the management cluster. Tekton has no production cluster credentials. After a successful pipeline, it writes a Git commit (updated image tag). ArgoCD picks up that commit and deploys. Tekton → Git → ArgoCD is the only path from code change to production.

**Sealed Secrets:** Asymmetric encryption for secrets in Git. The SealedSecret controller running in the cluster holds the private key. Engineers encrypt secrets with the public key using the `kubeseal` CLI. The resulting SealedSecret CRD is committed to Git safely. The controller decrypts to a Kubernetes Secret at reconciliation time. Plaintext secrets never appear in Git.

**Credential lifecycle:** Credentials (service accounts, API keys, certificates) have defined rotation schedules documented as governance artifacts. Rotation is a Git operation: update the credential, re-seal, commit. AWX can be triggered to rotate credentials on schedule via job templates. No credential has an infinite lifetime by default.

## Alternatives Considered

**Ansible Tower (licensed) instead of AWX** — commercial version of AWX. Rejected: AWX is the open-source upstream with identical features. University budget does not justify the licensing cost.

**Pulumi instead of Terraform** — infrastructure as code in general-purpose programming languages. Rejected: Terraform HCL is the team's existing language; the vSphere Terraform provider is mature; Pulumi's state management adds complexity without benefit at current scale.

**Jenkins for CI** — established CI platform. Rejected: Jenkins requires cluster credentials for deployment; Tekton runs in-cluster with Kubernetes-native RBAC; push-based deployment from Jenkins violates the GitOps authority model. Jenkins is appropriate for artifact CI; it is not appropriate as a CD tool in a GitOps platform.

**HashiCorp Vault for secrets** — dedicated secret store with dynamic secret generation. Rejected for Phase A/B: Vault is operationally significant — it requires HA, unseal procedures, and its own backup strategy. Sealed Secrets solves the GitOps secret problem with minimal operational overhead. Vault is the right answer when dynamic secrets (database credentials with TTL) are required. Revisit at Phase C.

## Trade-offs

**Accepted:** AWX as a dependency for configuration execution. If AWX is unavailable, no Ansible jobs run — OS updates, config changes, and runbook automation pause. Existing infrastructure continues operating. Mitigation: AWX HA deployment.

**Accepted:** Sealed Secrets key management. The SealedSecret controller private key is the encryption root. Key loss means existing SealedSecrets cannot be decrypted. Mitigation: private key backup procedure documented as a disaster recovery artifact.

**Accepted:** Tekton learning curve. Tekton's CRD-based pipeline model (Pipeline, Task, PipelineRun) is more complex than declarative CI YAML. Engineers must understand Tekton primitives. Mitigation: pipeline templates provided for common patterns.

**Gained:** Complete execution audit trail. Terraform runs in AWX or CI — logged. Ansible runs in AWX — logged, with job output stored. Tekton runs in-cluster — logged via Loki. Every execution event is observable.

**Gained:** Clean credential boundary between CI and CD. Tekton cannot access production. Production deployments only happen through ArgoCD reconciliation from Git. The credential surface for production is Sealed Secrets in Git + ArgoCD service accounts — not every engineer's laptop and CI system.

## Consequences

- `terraform apply` from an engineer laptop is prohibited; all applies run in AWX or Tekton
- `ansible-playbook` from an engineer laptop is prohibited; all runs go through AWX job templates
- All Tekton pipeline definitions live in the GitOps repo — no pipeline configuration outside Git
- Every secret in the platform is a SealedSecret or a dynamically generated credential (future: Vault)
- Credential rotation schedules are documented; rotation is a Git commit + AWX job, not a manual process
- Vault is explicitly deferred to Phase C; Sealed Secrets is not a temporary solution

## Evidence

- IMPL-0011, -0012, -0013, -0015, -0018 accepted Phase A
- IMPL-0030 (execution tooling strategy, Phase B) confirms Terraform + Ansible + Tekton as the execution stack
- Vault deferral is explicit in IMPL-0030 scope statement

## Principal-Level Signal

Execution tooling is where governance either holds or collapses. You can have perfect ADRs and a pristine GitOps repo and still have engineers running `kubectl` directly, `ansible-playbook` from laptops, and `terraform apply` with local state files. The governance is only as strong as the execution controls. Prohibiting ad-hoc execution is not a restriction on engineers — it is the mechanism that makes the audit trail real. AWX as the only Ansible interface, Tekton as the only path from code to production, Terraform in controlled execution — these are not tool choices. They are enforcement points.
