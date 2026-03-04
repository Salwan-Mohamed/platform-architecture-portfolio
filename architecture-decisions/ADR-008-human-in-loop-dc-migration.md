# ADR-008: Human-in-Loop Governance for Datacenter Migration

**Status:** Accepted  
**Date:** 2026-02-23  
**Decider:** Salwan Mohamed  
**Project:** DC Migration Platform  

---

## Context

Datacenter migration programs fail at rates that are well-documented in the industry. The most common failure modes are not technical — they are governance failures: decisions made without sufficient authority, changes executed without documented approval, assumptions treated as requirements, and risk accepted implicitly rather than explicitly.

The DC Migration Platform was designed as the governance system for a complex datacenter migration involving multiple workload owners, vendor dependencies, and phased cutover events. The question was: **how much automation is appropriate, and where must a human be in the decision loop?**

The context was complicated by the availability of AI tooling (Claude Code) that could theoretically execute many tasks autonomously. The risk was that automation would create the appearance of governance without the substance — generating documents, approving them, and executing changes all without meaningful human oversight.

---

## Constraints

| Constraint | Impact |
|------------|--------|
| Multiple workload owners with different risk tolerances | Cannot have a single automated decision authority for cross-owner decisions |
| Regulatory requirements for change documentation | Approvals must be traceable to named humans, not AI agents |
| Irreversibility: some migration steps cannot be rolled back | Conservative governance required for high-blast-radius actions |
| Compressed timeline: urgency pressure will push for shortcuts | Governance model must be lightweight enough to survive under pressure |
| AI tooling available and capable | Must explicitly define AI role to prevent governance theater |

---

## Decision

**All decisions rated P1 (program-level) or P2 (architecture-level) require explicit human approval from a named, role-appropriate authority. AI agents (Claude, Claude Code) are Advisory and Execution roles only — they draft, analyze, and execute on instruction; they never approve.**

Role model:
- **Human Owner (Sponsor):** Final authority on P1/P2 decisions; risk acceptance; phase gate sign-off
- **PM:** Approves timeline and resource changes; owns RACI
- **Claude (AI Advisor):** Drafts documents, analyzes gaps, proposes options with trade-offs. RECOMMEND only.
- **Claude Code (Automation):** Executes scripted tasks under PM direction. EXECUTE only; outputs validated by human before merge.

Core principle: **Evidence over assertion. Every claim links to evidence. Nothing is agreed unless it is documented.**

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|----------------|
| **Fully automated migration with AI approval** | Violates human accountability requirement; regulatory non-compliance; no meaningful risk acceptance |
| **ITSM ticket system as sole governance** | Tickets record intent, not outcomes; no evidence standard; changes can be executed without proper approval if ticket is "closed" |
| **Traditional project governance only (no AI)** | Documentation velocity would be too low; AI dramatically accelerates artifact generation without replacing human judgment |
| **Informal human oversight (verbal approvals)** | Non-auditable; known failure mode for migration programs; creates post-incident liability |

---

## Trade-offs

**What we gained:**
- Every P1/P2 decision has a named human owner and documented rationale
- AI accelerates documentation generation without replacing accountability
- Phase gates cannot be bypassed — exit criteria are explicit and signed off
- Evidence standard makes the program auditable from day one
- Rollback is always a documented option — not an afterthought

**What we gave up:**
- Decision velocity is constrained by human availability for approvals
- Framework requires discipline from PMs to enforce the process under timeline pressure
- AI capabilities are intentionally bounded — some efficiency is sacrificed for accountability

---

## Consequences

- DC Migration Platform repository implements 6 governance documents: RACI, PHASE-GATES, DECISION-LOG, DECISION-SEVERITY, CHANGE-CONTROL, APPROVAL-REGISTRY
- All merges to `main` require PM approval via pull request
- Branch strategy enforces governance: `main` (approved only), `draft/*` (WIP), `evidence/*` (raw intake)
- AI role is explicitly documented in the repository README — Recommendation authority only
- The pattern has been validated: zero unauthorized changes to production systems during migration planning phase

---

## Evidence

- [dc-migration-platform](https://github.com/Salwan-Mohamed/dc-migration-platform) — Full governance framework implementation
- [dc-migration-platform PHASE-GATES.md](https://github.com/Salwan-Mohamed/dc-migration-platform/blob/main/docs/01-governance/PHASE-GATES.md) — Phase gate definitions
- Core ADR-0088: AI Is Not an Actor
- Core ADR-0126: No Implicit Authority Infrastructure
- Platform Operating System: Human Owner as Sole Decision Authority

---

## Principal-Level Signal

> The most dangerous governance failure is the one that looks like governance but isn't. Auto-generated approvals, AI-signed documents, and silent PR merges create the appearance of oversight without the substance. For irreversible operations — and datacenter migrations are full of them — the question is not "can we automate this approval?" but **"who is personally accountable for this decision, and can we reach them at 2am if it goes wrong?"**
