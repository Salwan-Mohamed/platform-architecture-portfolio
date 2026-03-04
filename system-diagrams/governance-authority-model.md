# System Diagram: Governance Authority Model

> Who can decide what, at each layer of the Nexus Platform System.

## Authority Hierarchy

```mermaid
graph TB
    HO["🧑 Human Owner\nSole Decision Authority\n\nApproves phase transitions\nAccepts risk explicitly\nCloses all decisions"] 
    
    STOP["🛑 Stop Authority Hierarchy\n(Non-Negotiable Precedence)\n\n1. Production Operations\n2. Governance & Risk\n3. Platform Engineering\n4. Evolutionary Control\n5. TPO"]
    
    subgraph AGENTS["AI Agent Roles"]
        CLAUDE["Claude (AI Advisor)\nRECOMMEND only\n\nDrafts documents\nAnalyzes gaps\nProposes options"]
        CLAUDE_CODE["Claude Code (Automation)\nEXECUTE under instruction\n\nRuns scripts\nGenerates artifacts\nNever self-approves"]
    end
    
    subgraph GIT["Git Control Plane"]
        BASELINE["Baseline Automation\n→ Infrastructure"]
        RUNTIME["Runtime Automation\n→ Operations (24h SLA)"]
        AI_ADVISORY["AI Advisory\n→ Advisory ONLY"]
    end
    
    HO --> STOP
    HO --> AGENTS
    HO --> GIT
    
    style HO fill:#3b82f6,color:#fff
    style STOP fill:#ef4444,color:#fff
    style CLAUDE fill:#8b5cf6,color:#fff
    style CLAUDE_CODE fill:#8b5cf6,color:#fff
```

---

## Core / Product Authority Boundary

```mermaid
graph LR
    subgraph CORE["Nexus Platform Core\n(Immutable)"]
        PRINC["Principles"]
        ADR["183 ADRs"]
        GOV["Governance Models"]
        PHASES["Phase Definitions"]
        CAP["Capability Contracts"]
    end
    
    subgraph PRODUCT["Product Implementation\n(Disposable)"]
        VENDORS["Vendor Selections"]
        CONFIG["Tool Configurations"]
        ENV["Environment Specifics"]
        OPS["Operational Procedures"]
        MANIFESTS["Deployment Manifests"]
    end
    
    CORE -->|"Authority flows ONE WAY"| PRODUCT
    PRODUCT -.->|"Cannot override Core"| CORE
    
    style CORE fill:#1e40af,color:#fff
    style PRODUCT fill:#065f46,color:#fff
```

---

## Decision Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Intake: Decision Required
    
    Intake --> Validation: Enter through defined intake point
    Intake --> Rejected: Invalid entry (verbal, email, chat)
    
    Validation --> AuthorityCheck: Submit
    AuthorityCheck --> SignalCheck: Authority verified
    AuthorityCheck --> Rejected: No named authority
    
    SignalCheck --> FitnessAssign: Signals present
    SignalCheck --> Rejected: No signal reference
    
    FitnessAssign --> HumanReview: Fitness functions assigned
    
    HumanReview --> Accepted: Human Owner approves
    HumanReview --> Deferred: Awaiting external input
    HumanReview --> Rejected: Validation failed
    
    Accepted --> [*]
    Rejected --> [*]
    Deferred --> HumanReview: Input received
```

---

## DC Migration Governance Model

```mermaid
graph TB
    subgraph P1["P1 Decisions (Sponsor Required)"]
        SCOPE["Scope Changes"]
        BUDGET["Budget Changes"]
        GATES["Phase Gate Transitions"]
        RISK["Risk Acceptance"]
    end
    
    subgraph P2["P2 Decisions (Architect Required)"]
        ARCH["Architecture Changes"]
        VENDOR["Vendor Selection"]
        CUTOVER["Cutover Sequencing"]
    end
    
    subgraph P3["P3 Decisions (PM Discretion)"]
        SCHED["Schedule Adjustments"]
        RUNBOOK["Runbook Updates"]
        COMM["Communication Plans"]
    end
    
    SPONSOR["Human Owner\n(Sponsor)"] --> P1
    ARCHITECT["Platform Architect\n(Salwan)"] --> P2
    PM["Project Manager"] --> P3
    
    style P1 fill:#dc2626,color:#fff
    style P2 fill:#d97706,color:#fff
    style P3 fill:#16a34a,color:#fff
```
