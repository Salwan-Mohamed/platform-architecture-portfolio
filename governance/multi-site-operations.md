# Multi-Site Operations Model

> How the Sinai University platform is operated across 3 campus sites with a team of 1 principal + 3 site ops.

---

## Operating Model Principles

1. **Central governance, distributed operation.** Platform principles, ADRs, and change control are centralized. Day-to-day operations are distributed to site ops teams within defined guardrails.
2. **GitOps enforces consistency.** All sites share the same Git repository as the source of truth. Site-specific configuration is expressed as values overrides, not separate codebases.
3. **Automation is the equalizer.** Site ops teams don't need to be Kubernetes experts if the automation abstracts the complexity. AWX runbooks handle the majority of operational tasks.
4. **Observability is non-negotiable.** Every site has full observability coverage before any capability is released to that site. "We'll add monitoring later" is a blocked statement.

---

## Responsibility Matrix

| Domain | Principal Platform Engineer | Site Ops (per site) | Automated |
|--------|----------------------------|---------------------|----------|
| Platform architecture | Owner | Informed | N/A |
| ADR authoring | Owner | Consulted | N/A |
| Network policy (FortiManager) | Owner | Operator | Drift detection |
| Aruba switch config | Owner | Operator | AWX playbooks |
| Kubernetes cluster lifecycle | Owner | Viewer | CAPV + ArgoCD |
| Application deployment | Owner | N/A | ArgoCD GitOps |
| Monitoring dashboards | Owner | Operator | Auto-provisioned |
| Incident response | Lead | First responder | Alert routing |
| Change control | Approver | Requestor | PR automation |
| Runbook execution | N/A | Operator | AWX |

---

## Change Control Flow

```
Change Request (via PR)
        │
        ▼
   Principal Review
   (platform impact?)
        │
   ┌────┴────┐
   │         │
  Yes        No
   │         │
   ▼         ▼
  ADR?     PM Approve
   │         │
  Yes        ▼
   │      Merge to main
   ▼         │
  ADR Review  ▼
   │      ArgoCD sync
   ▼
  Accepted?
   │
  Yes
   │
   ▼
  Merge to main
   │
   ▼
  ArgoCD sync
   │
   ▼
  Drift check
```

---

## Incident Response Model

### Severity Classification

| Severity | Definition | Response SLA | Escalation |
|----------|-----------|-------------|------------|
| P1 | Multi-site impact; core services down | 15 min acknowledge | Principal + Leadership |
| P2 | Single-site impact; non-critical services | 1 hour acknowledge | Site Ops + Principal |
| P3 | Degraded performance; no outage | 4 hour acknowledge | Site Ops |
| P4 | Monitoring alert; no user impact | Next business day | Site Ops |

### Runbook Automation

All P3 and P4 responses have AWX runbooks. Site ops execute runbooks; they don't write fix scripts ad-hoc.

```bash
# AWX runbook execution examples
awx job_template launch --name "restart-argocd-sync"
awx job_template launch --name "check-vpn-tunnel-status"
awx job_template launch --name "rotate-service-account-credentials"
```

---

## Site Autonomy Boundaries

Site ops can do **without principal approval:**
- Execute AWX runbooks (approved list)
- Acknowledge monitoring alerts and follow runbook
- Update site-specific documentation
- Escalate to principal for P1/P2 incidents

Site ops **must have principal approval for:**
- Any network policy change (FortiGate or Aruba)
- Any Kubernetes workload change outside GitOps
- Any IAM/AD change
- Any infrastructure provisioning (new VMs, new VLANs)

---

## Operational Metrics

| Metric | Target | Current |
|--------|--------|---------|
| Mean Time to Detect (MTTD) | < 5 min | Prometheus alerting active |
| Mean Time to Acknowledge (MTTA) | P1: 15 min | Alert routing configured |
| Change success rate | > 95% | GitOps enforces pre-merge validation |
| Unauthorized changes | 0 | GitOps + drift detection |
| ADR currency | 100% documented decisions | 8 portfolio ADRs + 183 Core ADRs |
