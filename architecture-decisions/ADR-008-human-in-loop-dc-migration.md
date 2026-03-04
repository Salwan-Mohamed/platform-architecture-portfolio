# ADR-008: Human-in-Loop Governance for Datacenter Migration

**Status:** Accepted  
**Date:** 2026-02-23  
**Decider:** Salwan Mohamed  
**Domain:** Governance / Operations  
**Project:** DC Migration Platform

---

## Context

Datacenter migrations fail most often not because of technical problems — but because of organizational problems:
- Decisions made in meetings with no documentation trail
- Approvals assumed rather than obtained
- Evidence not collected until after the fact, when it's too late
- Scope changes made without impact assessment
- "Someone said it was okay" as the justification for a critical infrastructure change

The DC Migration Platform must enforce governance as the default behavior, not as overhead added after delivery.

---

## Constraints

- University environment with distributed decision-making authority
- Sponsor approval required for all P1/P2 decisions
- Change management process has audit requirements
- Multiple workstreams (network, compute, storage, applications) operating concurrently
- AI assistance available (Claude Code) for documentation generation — but AI cannot be an authority

---

## Decision

**All migration decisions require explicit human authorization documented in the repository. AI agents (Claude, Claude Code) can DRAFT and RECOMMEND but cannot APPROVE, FINALIZE, or EXECUTE without named human sign-off. Nothing is agreed unless it is documented. Nothing is executed unless it is approved.**

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|-----------------|
| AI-driven autonomous migration decisions | AI does not have accountability; cannot accept organizational risk; no authority |
| Email-based approval process | No traceability in the repo; emails are lost; no version control; doesn't satisfy audit requirements |
| Meeting minutes as governance artifacts | Minutes are summaries, not decisions; often incomplete; no formal acceptance ceremony |
| Implicit approval (silence = consent) | Creates disputes post-incident; breaks down when people are busy; legally insufficient |
| Single approver for all decisions | Creates bottleneck; does not distribute knowledge; approval becomes rubber stamp |

---

## Trade-offs

**Given up:**
- Speed — explicit documentation and approval takes more time than verbal agreement
- Flexibility — every change, even small ones, requires documentation

**Gained:**
- Legal and audit protection: every decision has a named decider, date, and rationale
- Post-migration archaeology: if something breaks 6 months later, the decision trail exists
- Organizational trust: teams know that decisions were made deliberately, not carelessly
- Knowledge transfer: new team members can read the decision log and understand why things are the way they are
- AI accountability model: Claude generates, human approves. This is the only safe model for AI-assisted infrastructure work.

---

## Authority Model

| Role | Authority |
|------|----------|
| Human Owner (Sponsor) | APPROVE all P1/P2 decisions; phase gate sign-off |
| PM | APPROVE timeline and resource changes |
| Claude (AI Advisor) | RECOMMEND only; outputs are drafts until human-reviewed |
| Claude Code (Automation) | EXECUTE only under PM direction; all outputs human-validated before merge |

---

## Consequences

**Positive:**
- DC Migration Platform repository structure enforces governance as architecture: `docs/01-governance/` contains RACI, approval registry, phase gates, decision severity, change control
- Evidence requirements are explicit: every claim links to `docs/06-evidence/`
- Immutability of approvals: once approved, documents are versioned, not overwritten

**Negative:**
- Governance overhead is real — team must be trained to follow the process
- Risk of governance theater: if people sign off without reading, the process is formal but not effective
  - Mitigation: Decision severity rating ensures that not all decisions require sponsor-level approval; only P1/P2

---

## Evidence

- [dc-migration-platform](https://github.com/Salwan-Mohamed/dc-migration-platform) — Repository structure with governance as architecture
- `docs/01-governance/PHASE-GATES.md` — No phase transition without gate approval
- `docs/01-governance/DECISION-SEVERITY.md` — Risk-rated decision routing
- `docs/01-governance/APPROVAL-REGISTRY.md` — Named approvals with timestamps

---

## Operational Lesson

The most valuable governance artifact in a DC migration is the decision log, not the technical design documents. Technical designs are aspirational — they describe what you planned to do. The decision log describes what you actually decided and why. These two artifacts diverge constantly during execution. The decision log is the ground truth.

---

*This ADR reflects Principal-level thinking: designing the governance system, not just the technical system.*
