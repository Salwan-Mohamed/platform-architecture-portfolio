# System Diagram: Nexus Platform Full Stack

> Architecture-as-code diagram using Mermaid. Render in GitHub or any Mermaid-compatible viewer.

## Platform Layer Model

```mermaid
graph TB
    subgraph OS["Platform Operating System"]
        POS["Signal-Driven Governance\nAuthority Model\nDecision Gates\nFitness Functions"]
    end

    subgraph CORE["Nexus Platform Core (183 ADRs)"]
        PHIL["Platform Philosophy\nADR-0091"]
        PHASE["Phase Model A→B→C→D\nADR-0114"]
        GOV["Governance Model\nADR-0126"]
        NET["Network Capability Model\nADR-0141"]
        OBS["8-Axis Observability\nADR-0152"]
    end

    subgraph PRODUCT["Sinai University Product"]
        subgraph GITOPS["GitOps Layer"]
            ARGO["ArgoCD\nGitOps Reconciler"]
            TEKTON["Tekton\nCI Pipelines"]
            AWX["AWX\nRunbook Engine"]
        end

        subgraph K8S["Kubernetes Layer"]
            RKE2["RKE2\nCIS-Hardened"]
            CILIUM["Cilium\nNetwork Policy + eBPF"]
            CAPV["Cluster API (CAPV)\nLifecycle Management"]
        end

        subgraph IDENTITY["Identity Layer"]
            KEYCLOAK["Keycloak\nIdentity Broker"]
            AD["Active Directory\nIdentity Authority"]
        end

        subgraph VIRT["Virtualization Layer"]
            VSPHERE["VMware vSphere\nCompute Fabric"]
        end

        subgraph NET2["Network Layer"]
            FGT["FortiGate 1101E\n3 Sites"]
            ARUBA["Aruba Switching\nCampus Fabric"]
            VPN["Full-Mesh IPsec VPN\nBGP Routing"]
        end

        subgraph OBS2["Observability Layer"]
            PROM["Prometheus\nMetrics"]
            GRAF["Grafana\nVisualization"]
            LOKI["Loki\nLogs"]
        end
    end

    OS --> CORE
    CORE --> PRODUCT
    ARGO --> RKE2
    TEKTON --> RKE2
    AWX --> VSPHERE
    AWX --> FGT
    AWX --> ARUBA
    RKE2 --> VSPHERE
    CILIUM --> RKE2
    CAPV --> VSPHERE
    KEYCLOAK --> AD
    KEYCLOAK --> RKE2
    FGT --> VPN
    VPN --> ARUBA
    PROM --> RKE2
    PROM --> VSPHERE
    GRAF --> PROM
    GRAF --> LOKI
```

---

## Authority Flow Diagram

```mermaid
graph LR
    GIT["Git\n(Source of Truth)"] --> ARGO["ArgoCD\nReconciler"]
    GIT --> TF["Terraform\nInfrastructure"]
    GIT --> ANS["Ansible/AWX\nConfiguration"]
    
    ARGO --> K8S["Kubernetes State"]
    TF --> VSPHERE["vSphere VMs"]
    ANS --> NET["Network Devices"]
    
    HUMAN["Human Author\n(PR + Review)"] --> GIT
    
    AI["Claude Code\n(Advisory)"] -.->|"Drafts only\nNo approval"| GIT
    
    DRIFT["Drift Detection\n(ArgoCD/Terraform)"] -->|"Alert"| HUMAN
```

---

## Phase Maturity Model

```mermaid
graph LR
    A["Phase A\nInfrastructure Control\n✅ Complete"] --> B["Phase B\nExplainable Platform\n🟡 In Progress"]
    B --> C["Phase C\nPlatform Knowledge\n⚪ Planned"]
    C --> D["Phase D\nGoverned Self-Service\n⚪ Future"]
    
    style A fill:#22c55e,color:#fff
    style B fill:#f59e0b,color:#fff
    style C fill:#94a3b8,color:#fff
    style D fill:#94a3b8,color:#fff
```

---

## Multi-Site Topology

```mermaid
graph TD
    subgraph MAIN["Main Campus (Sinai)"]
        FGT_M["FortiGate 1101E\nActive-Passive HA"]
        ARUBA_M["Aruba Core Stack"]
        VSPHERE_M["vSphere Cluster\nRKE2 Control Plane"]
    end

    subgraph B1["Branch Site 1"]
        FGT_B1["FortiGate 1101E"]
        ARUBA_B1["Aruba Access Layer"]
        VSPHERE_B1["vSphere Cluster\nRKE2 Worker Nodes"]
    end

    subgraph B2["Branch Site 2"]
        FGT_B2["FortiGate 1101E"]
        ARUBA_B2["Aruba Access Layer"]
    end

    FGT_M <-->|"IPsec + BGP"| FGT_B1
    FGT_M <-->|"IPsec + BGP"| FGT_B2
    FGT_B1 <-->|"IPsec + BGP"| FGT_B2

    FMGR["FortiManager\nCentralized Policy"] --> FGT_M
    FMGR --> FGT_B1
    FMGR --> FGT_B2
```

---

## Observability Framework (8 Axes)

```mermaid
graph TB
    subgraph OBS["8-Axis Observability Framework"]
        AX1["Infrastructure\nCompute/Storage/Network Health"]
        AX2["Application\nApp Metrics + Traces"]
        AX3["Service Health\nSLOs + Error Budgets"]
        AX4["Incident\nMTTR + Escalation"]
        AX5["Portfolio\nService Catalog + Dependencies"]
        AX6["Platform\nGitOps Sync + Drift"]
        AX7["FinOps\nCost Allocation"]
        AX8["Business Operations\nKPIs + Capacity"]
    end

    PROM["Prometheus"] --> AX1
    PROM --> AX2
    PROM --> AX3
    LOKI["Loki"] --> AX4
    ARGO["ArgoCD"] --> AX6
    GRAF["Grafana"] --> AX1
    GRAF --> AX2
    GRAF --> AX3
    GRAF --> AX4
    GRAF --> AX5
    GRAF --> AX6
    GRAF --> AX7
    GRAF --> AX8
```
