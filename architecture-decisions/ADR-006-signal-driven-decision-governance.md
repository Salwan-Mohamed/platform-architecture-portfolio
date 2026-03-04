# ADR-006: Signal-Driven Decision Governance

**Status:** Accepted  
**Date:** 2026-01-10  
**Decider:** Salwan Mohamed  
**Project:** Platform Operating System  

---

## Context

As the Nexus Platform grew, a recurring problem emerged: architectural decisions were being made based on opinion, convenience, and recency bias rather than on observable evidence. A decision to adopt a new tool would be justified by "the team likes it" or "I saw it at a conference" — without any signal about whether it solved an actual, measured problem. Conversely, decisions to retire tooling were resisted based on "we've always done it this way" without any signal about whether the tooling was delivering value.

The platform also needed to resist the pressure of urgency: during incidents, the temptation to "just fix it" by bypassing governance was strong. Without a formal decision framework, these shortcuts accumulated into a governance debt that made the platform unpredictable.

---

## Constraints

| Constraint | Impact |
|------------|--------|
| Decisions made under pressure tend to bypass governance | Framework must be lightweight enough to use in urgency without abandoning it entirely |
| Platform has multiple stakeholders with competing priorities | Decisions must be traceable to evidence, not political alignment |
| AI tooling (Claude Code) is now part of the operational workflow | Must define explicitly what AI can and cannot decide |
| Ops team of 2-3 people must be able to operate the governance framework | Cannot require a dedicated governance analyst |
| Historical pattern of tribal knowledge and undocumented decisions | New framework must make implicit decisions explicit |

---

## Decision

**All architectural decisions must reference observable signals from a canonical Signals Registry. Decisions without signal references are rejected. Opinions do not override signals. AI agents are advisors only — they cannot approve or finalize decisions.**

Signal categories:
- **Correctness**: Policy enforcement outcomes, config drift measurements
- **Value**: Adoption rate, effort displacement, time-to-production delta  
- **Stability**: Error rate trends, incident frequency, rollback rate
- **Evolution**: Time-to-introduce-capability, coupling indicators, blast-radius

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|----------------|
| **Lightweight ADR-only model (no signals)** | ADRs document decisions but don't validate them against reality; produces governance theater |
| **ITSM-based change control** | Change tickets record intent, not outcomes; no feedback loop between decision and result |
| **AI-driven decision making** | Core principle violation: AI is not an actor (Nexus ADR-0088); AI advisory without signal validation creates hallucinated governance |
| **No formal model — experienced judgment** | Non-transferable; doesn't survive team turnover; produces the tribal knowledge problem it was designed to solve |

---

## Trade-offs

**What we gained:**
- Every decision is traceable to evidence — audit trail exists by construction
- Governance is objective: "the signals show X" is a more defensible position than "I think Y"
- AI agents (Claude Code) are bounded by the framework — they generate artifacts, not decisions
- The Signals Registry becomes organizational memory: what we measured, what we found, what we decided
- Decisions that age badly are visible: signals will show if the decision's consequences diverged from expectations

**What we gave up:**
- Decision velocity is slightly lower — must establish signal baseline before deciding
- Framework requires discipline to maintain the Signals Registry as new capabilities are added
- Some decisions are genuinely judgment-based (strategic, political) and don't fit neatly into signal validation

---

## Consequences

- Platform Operating System repository implements this model as its core operating contract
- Claude Code's role is explicitly defined: Operator, not Decider; Generator, not Approver
- Signals Registry is a canonical source in the Platform Operating System
- Phase 10 of the Platform OS is specifically "Signal Activation & Baseline Establishment" — you observe before you enforce
- DC Migration Platform adopts the same principle: evidence-over-assertion governance

---

## Evidence

- [platform-operating-system README](https://github.com/Salwan-Mohamed/platform-operating-system) — Full signal-driven governance model
- [dc-migration-platform governance](https://github.com/Salwan-Mohamed/dc-migration-platform/blob/main/docs/01-governance/) — Evidence-based migration governance
- Core ADR-0088: AI Is Not an Actor
- Core ADR-0091: Nexus Platform Philosophy
- Core ADR-0126: No Implicit Authority Infrastructure

---

## Principal-Level Signal

> The purpose of a decision framework is not to slow down decisions — it is to prevent the accumulation of invisible debt. Every undocumented decision is a landmine for the next person who inherits the system. Signal-driven governance is how you make your platform's decision history legible to anyone who joins the team in 3 years.
