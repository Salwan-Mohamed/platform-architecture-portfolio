# LinkedIn Post 04: Phase 1 Closed — What a Governed Platform Foundation Looks Like

**Target Audience:** Platform engineers, SREs, infrastructure architects, engineering managers  
**Angle:** The platform story — what was built, what it took, what honest closure means  
**Post length:** ~320 words

---

## Post Text

Phase 1 of the Nexus Platform at Sinai University is closed.

Not "mostly done." Not "good enough." Closed against 14 documented criteria, each with a traceable evidence pack.

Here's what that means in practice:

**What's running:**
- RKE2 Kubernetes cluster — 6 nodes, all Ready
- ArgoCD managing 18 applications — all Synced, all Healthy
- Keycloak federated to Active Directory — 1,255 users, LDAPS, no shadow stores
- SSO live across 5 platform consumers: ArgoCD, Grafana, Longhorn UI, Alertmanager UI, Hubble UI
- Full observability stack: Prometheus, Grafana, Loki, Alertmanager, Hubble
- Longhorn storage with MinIO S3 backup — backup chain operational, isolated restore drill proven
- Gateway API ingress with internal PKI — cert-manager issuing, trust-manager distributing
- Full-mesh IPsec VPN across 3 campus sites — no hub, no single point of failure

**What "closed" actually means:**
Every platform change went through a Pull Request. Every decision has an ADR. Every service has an SLO. The governance model is in the repository, not in someone's head.

When an AD LDAPS CA re-key incident hit during operations, we resolved it under the same governed evidence model — diagnosed, contained, and closed with a dated incident report. That's not a story about incident response tooling. It's a story about what a platform with a real governance model looks like when something goes wrong.

**What it's not:**
Not portable yet. Not security-hardened for production at scale. Not a commercial product. Those are Phase 2 problems — and I know exactly what they are because we documented them.

The honest boundary is the credential: a foundation reference implementation that does what it claims, knows what it doesn't, and has the evidence to tell the difference.

Phase 2 agenda is governed and recorded. Not opened yet.

The work is on GitHub.

---

## Diagram

```mermaid
graph TB
    subgraph CLOSED["Phase 1 — CLOSED ✓ (PR #97, 2026-05-31)"]
        subgraph K8S["Kubernetes Layer"]
            RKE2["RKE2 v1.34\n6 nodes Ready"]
            CILIUM["Cilium 1.15.6\neBPF CNI"]
            HUBBLE["Hubble\nNetwork flows"]
        end
        subgraph GITOPS["GitOps Layer"]
            ARGOCD["ArgoCD v2.13.3\n18 apps — Synced + Healthy"]
        end
        subgraph IDENTITY["Identity Layer"]
            KEYCLOAK["Keycloak 26.3.3\n1,255 AD users federated"]
            SSO["SSO: 5 consumers live\nClass A + Class C"]
        end
        subgraph OBS["Observability Layer"]
            PROM["Prometheus v2.53.1"]
            GRAFANA["Grafana v11.2.2"]
            LOKI["Loki v2.9.10"]
        end
        subgraph STORAGE["Storage Layer"]
            LONGHORN["Longhorn v1.6.2\nBackup → MinIO S3"]
        end
    end

    subgraph OPEN["Phase 2 — UNOPENED"]
        P2["Portability / Hardening\nAgenda governed, not started"]
    end

    CLOSED --> OPEN

    style CLOSED fill:#14532d,color:#fff
    style OPEN fill:#1e3a5f,color:#fff
```

---

## Notes for Human Review
- [ ] Confirm comfort with "not security-hardened" framing — accurate per source but direct
- [ ] The AD LDAPS incident reference is accurate (docs/reports/2026-06-03-keycloak-ad-ldaps-truststore-remediation-closure.md) — decide if you want to include or remove
- [ ] "Phase 2 agenda is governed and recorded. Not opened yet." — intentionally creates forward tension; adjust if preferred
- [ ] GitHub link: add your portfolio repo URL at the end
