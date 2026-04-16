# Discovery Report: nexus-platform-product-sinai-university

> **Agent:** Discovery Agent  
> **Date:** 2026-04-16  
> **Source repo:** Salwan-Mohamed/nexus-platform-product-sinai-university  
> **Portfolio gap this addresses:** IMPL-0021 through IMPL-0031 not yet reflected in portfolio ADRs

---

## What Changed Since Last Sync

The source repo now contains 11 approved IMPL-series ADRs (0021–0031), all dated 2026-02-04. These represent the Phase B architecture specification for the Sinai University platform — the layer between the vendor-agnostic Nexus Core (183 ADRs) and the actual infrastructure. The portfolio had none of these.

---

## IMPL ADR Inventory (0021–0031)

| ADR | Title | Status | Principal Signal |
|-----|-------|--------|------------------|
| IMPL-0021 | DNS Domain Strategy | APPROVED | `platform.su.intra` as the internal platform domain; split-horizon DNS |
| IMPL-0022 | Storage Class Strategy | APPROVED | Longhorn as default dynamic provisioner; storage class per workload tier |
| IMPL-0023 | OIDC Authentication | APPROVED | Keycloak brokers AD via LDAPS; no shadow identity stores |
| IMPL-0024 | Group-Based Authorization | APPROVED | AD groups → Keycloak → OIDC claims → platform RBAC; no local role management |
| IMPL-0025 | SLO Operating Model | APPROVED | SLOs are team contracts not monitoring alerts; Phase B defines baseline targets |
| IMPL-0026 | Alerting Strategy | APPROVED | Alert on SLO breach signals, not raw metrics; alert routing via Alertmanager |
| IMPL-0027 | Phase Transition Criteria | APPROVED | Hard gates with evidence requirements before Phase A→B→C→D advancement |
| IMPL-0028 | Infrastructure Execution Parameters | APPROVED | Node sizing, resource limits, and cluster parameters formally specified |
| IMPL-0029 | CNI / Network Stack | APPROVED | Cilium (eBPF) replaces Canal (default); Hubble required for network observability |
| IMPL-0030 | Execution Tooling Strategy | APPROVED | Ansible via AWX for all configuration; Terraform for infrastructure; no manual CLI |
| IMPL-0031 | North-South Exposure Model | APPROVED | Nginx Ingress as primary; Gateway API path defined for future |

---

## Architecture Signals Extracted

### Identity Layer (IMPL-0023, IMPL-0024)
- **Keycloak** deployed at `sso.platform.su.intra` as the single OIDC provider
- **Active Directory** (`su.intra`, DCs: KANSYSDC01/02) is the only identity source — no local Keycloak users
- **LDAPS** (port 636) mandatory; plain LDAP forbidden
- **Token lifetimes**: Access 5m, Refresh 30m — security-first, not convenience-first
- **Group chain**: AD groups → Keycloak LDAP federation → OIDC claims → platform RBAC
- All platform services (ArgoCD, Grafana, Alertmanager) are OIDC relying parties with no local auth fallback

### Network Layer (IMPL-0029, IMPL-0031)
- **Cilium** replaces Canal as CNI — eBPF-based, L7-aware, not iptables
- **Hubble** required alongside Cilium for network flow observability
- Canal explicitly superseded — this is an architectural amendment, not a default swap
- **Nginx Ingress** for North-South traffic today; Gateway API as the defined future path
- Service mesh (Linkerd/Istio) explicitly excluded — Cilium L7 policy deemed sufficient
- OpenTelemetry explicitly deferred — Prometheus + Loki sufficient for initial platform baseline

### Observability Layer (IMPL-0025, IMPL-0026)
- SLOs are **team contracts** — defined per service before any alerting is configured
- Alerting fires on SLO burn rate signals, not raw metric thresholds
- Alertmanager handles routing; Prometheus handles evaluation
- Hubble adds network-layer dimension to observability model

### Execution Layer (IMPL-0028, IMPL-0030)
- Ansible + AWX for all configuration execution — no manual CLI permitted
- Terraform for infrastructure provisioning
- Node sizing formally specified (the specific values are in IMPL-0028)
- All execution tooling choices made before Phase B implementation begins

### DNS Layer (IMPL-0021)
- Internal platform domain: `platform.su.intra`
- Split-horizon DNS: internal services resolve via internal DNS; external via public DNS
- All platform service FQDNs follow pattern: `<service>.platform.su.intra`

### Storage Layer (IMPL-0022)
- Longhorn as default dynamic storage provisioner
- Storage classes defined per workload tier (not one-size)

### Governance Layer (IMPL-0027)
- Phase transitions require hard evidence gates — not calendar-based
- Each phase (A→B→C→D) has explicit documented advancement criteria

---

## Portfolio ADRs to Generate

Based on the above signals, recommend generating these portfolio-level ADRs:

| Portfolio ADR | Source IMPL(s) | Topic |
|---------------|----------------|-------|
| ADR-009 | IMPL-0023 + IMPL-0024 | Keycloak as Identity Broker (AD → OIDC) |
| ADR-010 | IMPL-0029 | Cilium CNI over Canal Default |
| ADR-011 | IMPL-0025 + IMPL-0026 | SLO-Driven Alerting Model |
| ADR-012 | IMPL-0027 | Evidence-Gated Phase Transitions |

---

## Gaps Identified

1. **IMPL-0022 (Longhorn storage)** — no portfolio ADR yet; recommend ADR-013
2. **IMPL-0030 (Execution tooling)** — reinforces ADR-002 (GitOps) with specifics; update ADR-002 consequences section
3. **IMPL-0031 (North-South exposure)** — recommend ADR-014 on Ingress/Gateway API strategy
4. **CS-01** — needs updating; CNI decision (Cilium) and OIDC architecture were not in the original case study
5. **Diagrams** — no portfolio diagram shows the Keycloak identity chain or Cilium/Hubble observability layer

---

## Files to Read for Full Context

- `docs/adr/IMPL-0023-oidc-authentication.md` — most detailed; 14KB, full auth flow diagrams
- `docs/adr/IMPL-0024-group-based-authorization.md` — 26KB; complete RBAC model
- `docs/adr/IMPL-0026-alerting-strategy.md` — 28KB; full alerting rule taxonomy
- `docs/CORE-SYNC-REPORT-2026-02-02.md` — 18KB; Core v14 → Product sync state
