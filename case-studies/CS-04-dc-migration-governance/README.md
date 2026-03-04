# Case Study 04: Governance-First Datacenter Migration Platform

> **Role:** Platform Architect + Migration Governance Designer  
> **Environment:** Multi-workload datacenter migration program  
> **Duration:** 2026 Q1 → Q2  
> **Complexity:** ⭐⭐⭐⭐  

---

## The Problem

Datacenter migration programs are where infrastructure projects go to fail at scale. The failure modes are well-known: decisions made without authority, changes executed without documentation, risk accepted implicitly by whoever happened to be present, and "we'll document it later" becoming the permanent state.

The specific challenge: design a migration governance platform that could handle a complex, multi-workload datacenter migration with multiple stakeholders, vendor dependencies, and irreversible cutover events — while incorporating AI tooling (Claude Code) in a way that accelerated work without compromising accountability.

The question that drove the design: **"Who is personally accountable for each decision, and can we reach them at 2am if it goes wrong?"**

---

## The Environment

| Dimension | Detail |
|-----------|--------|
| **Program scope** | Multi-workload DC migration (physical + virtual) |
| **Stakeholders** | Sponsor, PM, workload owners, vendors, ops team |
| **Decision types** | Strategic, architectural, operational, vendor |
| **AI tooling** | Claude + Claude Code active throughout |
| **Compliance requirement** | All decisions must be traceable and approved by named humans |
| **Timeline** | Phased migration over 12 weeks |

---

## The Constraints

- **Irreversibility:** Some migration steps (decommission legacy systems) cannot be rolled back — conservative governance mandatory
- **Urgency pressure:** Timeline creates constant temptation to bypass approval processes
- **Multiple approval authorities:** Different decisions require different approvers — no single person can approve everything
- **AI availability:** Claude Code is capable of generating and "approving" artifacts autonomously — must explicitly prevent governance theater
- **Evidence standard:** Every claim must link to evidence — "I remember the vendor said..." is not acceptable

---

## The Architecture

### Design Principle: Evidence over Assertion

The platform is built on a single principle: nothing is agreed unless it is documented, nothing is documented unless it links to evidence, and nothing is approved unless it is signed off by a named human with the appropriate authority.

### Governance Document Set

```
dc-migration-platform/docs/01-governance/
├── RACI.md                 ← Who is responsible, accountable, consulted, informed
├── PHASE-GATES.md          ← What must be true before each phase starts
├── DECISION-LOG.md         ← All decisions with date, decider, rationale
├── DECISION-SEVERITY.md    ← P1/P2/P3 classification with approval authority
├── CHANGE-CONTROL.md       ← How changes are proposed, reviewed, approved
└── APPROVAL-REGISTRY.md    ← Signed approvals with name, date, scope
```

### Role Model (Human vs AI)

```
┌────────────────────────────────────────────────────────┐
│                   AUTHORITY MODEL                      │
├────────────────────┬───────────────────────────────────┤
│  Human Owner       │ APPROVE all P1/P2 decisions        │
│  (Sponsor)         │ Sign phase gate transitions        │
│                    │ Accept risk formally               │
├────────────────────┼───────────────────────────────────┤
│  PM                │ APPROVE timeline/resource changes  │
│                    │ Own RACI and communication plan    │
├────────────────────┼───────────────────────────────────┤
│  Claude            │ RECOMMEND only                     │
│  (AI Advisor)      │ Draft documents, analyze gaps      │
│                    │ Propose options with trade-offs    │
├────────────────────┼───────────────────────────────────┤
│  Claude Code       │ EXECUTE under PM direction         │
│  (Automation)      │ Inventory parsing, config diffing  │
│                    │ All outputs validated before merge │
└────────────────────┴───────────────────────────────────┘
```

### Phase Gate Model

Each phase transition requires:
1. Exit criteria documented and verified
2. Sponsor sign-off (for P1 gates)
3. Evidence linked (not asserted)
4. Rollback option defined
5. Communication plan updated

### Branch Strategy Enforces Governance

| Branch | Purpose | Merge Authority |
|--------|---------|----------------|
| `main` | Approved, baselined artifacts only | PM via PR |
| `draft/*` | WIP documents under review | PR to main |
| `evidence/*` | Raw evidence intake before cataloging | PM cataloging |

---

## The Key Decision Points

### Decision 1: AI as Advisor, Not Authority
The most critical design decision was defining Claude's role explicitly before the program started. Without this definition, AI-generated documents get treated as approved artifacts by whoever reads them next.

The explicit rule: **"Claude drafts; humans approve. No artifact is final until a named human signs off."** This is written in the repository README, not in a separate policy document that no one reads.

**Why it mattered:** During execution, there were multiple instances where AI-generated analysis was high quality and tempting to treat as final. The explicit role definition prevented governance shortcuts even under timeline pressure.

### Decision 2: Decision Severity Classification
Not all decisions require the same approval authority. The DECISION-SEVERITY.md document classifies decisions as P1 (sponsor required), P2 (architect required), or P3 (PM discretion). This prevents two failure modes: approving trivial decisions at too high a level (bottleneck) and approving strategic decisions at too low a level (governance failure).

### Decision 3: Evidence in the Repository
All evidence — vendor emails, configuration snapshots, screenshots, signed approvals — is stored in the repository under `docs/06-evidence/`. This means the repository IS the audit package at program close. No separate audit assembly required.

---

## The Outcome

- **Zero unauthorized production changes** during the planning phase
- **100% decision traceability:** Every P1/P2 decision has a named human owner, date, and rationale
- **Phase gate integrity:** No phase transition made without documented exit criteria and sponsor sign-off
- **AI role clarity:** No governance theater — AI outputs are marked as drafts until human review
- **Audit-ready by default:** The repository IS the audit package — no separate assembly at program close

---

## Operational Lessons

**1. The governance framework must be in the repository, not in a separate wiki.**  
Governance documents that live in Confluence or SharePoint become stale and ignored. When the RACI and PHASE-GATES documents are in the same repository as the work, they are read and updated as part of the normal workflow.

**2. Decision severity classification is the highest-ROI governance document.**  
Knowing which decisions need which approval level prevents both bottlenecks (everything goes to the sponsor) and governance failures (strategic decisions decided by the wrong authority).

**3. "Evidence over assertion" is a cultural change, not a technical change.**  
The hardest part of the evidence requirement is getting stakeholders to understand that "I remember the vendor confirmed this" is not evidence. It requires consistent enforcement: every claim must link to something in `docs/06-evidence/`.

**4. AI governance theater is a real risk.**  
When AI tooling is involved in document generation, there is a natural tendency to treat AI outputs as more authoritative than they are. The antidote is explicit role documentation and a culture of "who approved this?" as the first question when any artifact is reviewed.

---

## Source Repositories

| Repository | Role |
|------------|------|
| [dc-migration-platform](https://github.com/Salwan-Mohamed/dc-migration-platform) | Full governance framework |
| [platform-operating-system](https://github.com/Salwan-Mohamed/platform-operating-system) | Governance principles source |
