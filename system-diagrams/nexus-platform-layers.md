# System Diagram: Nexus Platform Layers
## Sinai University Implementation

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  LAYER 5: GOVERNANCE                                                         │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌────────────────┐  │
│  │ Nexus Core      │ │ 183 ADRs        │ │ Platform OS     │ │ Phase Gates    │  │
│  │ (doctrine)      │ │ (decisions)     │ │ (enforcement)   │ │ (advancement)  │  │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘ └────────────────┘  │
│                                                                              │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  LAYER 4: OBSERVABILITY (8 Axes)                                             │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌────────┐ ┌────────┐  │
│  │ Infra   │ │ App     │ │ Svc     │ │ Incident│ │ Portfolio│ │ Platform│ │ FinOps │ │ BizOps │  │
│  │ Health  │ │ Metrics │ │ Health  │ │ Track   │ │ Catalog  │ │ Sync    │ │ Cost   │ │ KPIs   │  │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘ └────────┘ └────────┘  │
│         Prometheus + Grafana + Loki                                          │
│                                                                              │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  LAYER 3: WORKLOAD / KUBERNETES                                              │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │  RKE2 (Kubernetes Distribution)                                        │  │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐  │  │
│  │  │ ArgoCD      │ │ Cilium       │ │ Tekton       │ │ Cluster API │  │  │
│  │  │ (GitOps)    │ │ (eBPF CNI)   │ │ (Pipelines)  │ │ CAPV (vSphere│  │  │
│  │  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘  │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  LAYER 2: VIRTUALIZATION + IDENTITY                                         │
│  ┌───────────────────────┐ ┌────────────────────────────────────────┐  │
│  │ VMware vSphere         │ │ Active Directory → Keycloak          │  │
│  │ (Virtualization Layer) │ │ (Legacy IdP)     (OIDC Broker)       │  │
│  │ CAPV manages k8s nodes │ │                                         │  │
│  └───────────────────────┘ └────────────────────────────────────────┘  │
│                                                                              │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  LAYER 1: NETWORK FOUNDATION                                                │
│  ┌──────────────────┐ ┌──────────────────┐ ┌────────────────────────────┐  │
│  │ FortiGate 1101E  │ │ Full-mesh IPsec  │ │ Aruba campus switching         │  │
│  │ (Edge firewall   │ │ (Kantra↔Arish↔  │ │ AWX-automated Ansible          │  │
│  │  at each site)   │ │  Katamia)        │ │ VLAN segmentation per site     │  │
│  └──────────────────┘ └──────────────────┘ └────────────────────────────┘  │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

## Key Architectural Properties

| Property | Implementation |
|----------|---------------|
| **GitOps-first** | All state declared in Git; ArgoCD reconciles continuously |
| **Zero-trust boundary** | Cilium network policy; Keycloak OIDC; FortiGate UTM |
| **Branch autonomy** | Full-mesh VPN; local internet breakout; no datacenter dependency |
| **Phase-gated** | Phase A active; Phase B partial; advancement requires gate approval |
| **Evidence-based** | 197 ADRs; all configuration in version control |
| **Observable** | 8-axis observability framework: Prometheus + Grafana + Loki |

## Mermaid Diagram

```mermaid
graph TB
    subgraph GOV["Layer 5: Governance"]
        CORE["Nexus Core\n183 ADRs"]
        POS["Platform OS\nSignal-driven"]
        PHASES["Phase Gates\nA/B/C/D"]
    end

    subgraph OBS["Layer 4: Observability"]
        PROM["Prometheus"]
        GRAF["Grafana"]
        LOKI["Loki"]
    end

    subgraph K8S["Layer 3: Kubernetes"]
        RKE2["RKE2"]
        ARGO["ArgoCD"]
        CIL["Cilium"]
        CAPV["Cluster API (CAPV)"]
    end

    subgraph VIRT["Layer 2: Virtualization + Identity"]
        VSPHERE["VMware vSphere"]
        AD["Active Directory"]
        KC["Keycloak"]
    end

    subgraph NET["Layer 1: Network"]
        FG["FortiGate 1101E x3"]
        VPN["Full-mesh IPsec"]
        ARUBA["Aruba Switching"]
    end

    GOV --> OBS
    OBS --> K8S
    K8S --> VIRT
    VIRT --> NET

    AD -->|OIDC bridge| KC
    KC --> RKE2
    CAPV --> VSPHERE
    ARGO --> RKE2
    FG <-->|IPsec mesh| FG
```
