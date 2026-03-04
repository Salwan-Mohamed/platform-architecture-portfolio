# Case Study 04 — Datacenter Migration Platform

> *Building a governance-first migration program: nothing agreed unless documented, nothing executed unless approved*

**Repository**: [dc-migration-platform](https://github.com/Salwan-Mohamed/dc-migration-platform) (private)  
**Status**: Framework complete · Execution pending  
**Philosophy**: Evidence over assertion · Human-gated authority · Traceability by default

---

## Problem

Datacenter migrations fail in predictable ways:
1. **Scope creep without governance** — things get migrated that shouldn't be, or things get skipped
2. **Verbal agreements that evaporate** — "we decided in the meeting" leads to contradictory execution
3. **Evidence gaps during cutover** — can't prove what state systems were in before migration
4. **No rollback clarity** — when things go wrong, nobody knows what "back to working state" means

The challenge: build a migration governance system that prevents these failure modes before they happen.

---

## Environment

- **Program type**: Datacenter consolidation / migration for multi-site organization
- **Stakeholder complexity**: Executive sponsors, project managers, technical leads, operations staff, vendors
- **Risk profile**: Production workloads — failure has business impact
- **Constraint**: Multiple external vendors involved in execution
- **Tool**: GitHub as single source of truth — documentation as code

---

## Constraints

- **No verbal agreements**: Every decision must be documented; nothing is agreed until written
- **Evidence trail for audits**: All configuration snapshots, vendor communications, approvals must be preserved
- **Multi-role governance**: Sponsor, PM, AI advisor, and automation each have defined and bounded authority
- **Phase-gate discipline**: No phase can begin without documented exit criteria from the previous phase

---

## Architecture Decision

### Four-Role Authority Model

The defining architectural decision: **explicitly bound the authority of every participant including AI tools**.

| Role | Authority |
|------|-----------|
| Human Owner (Sponsor) | APPROVE — all P1/P2 decisions, gate sign-off |
| PM | APPROVE — schedule and resource changes |
| Claude (AI) | RECOMMEND ONLY — drafts, analysis, gap identification |
| Claude Code | EXECUTE ONLY — under PM direction, outputs validated before merge |

**Rules**:
- Claude drafts; humans approve
- No artifact is final until a named human signs off
- AI cannot self-approve any artifact

### Phase-Gate Structure

```
Phase 1: Discovery
  Entry: Program charter signed
  Exit:  As-is inventory complete + validated
  Gate:  PM approval + Sponsor acknowledgment

Phase 2: Design
  Entry: Phase 1 gate passed
  Exit:  Target architecture approved + delta plan complete
  Gate:  Technical lead approval + Sponsor sign-off

Phase 3: Execution
  Entry: Phase 2 gate passed
  Exit:  All workloads migrated + smoke tests passed
  Gate:  PM approval per wave

Phase 4: Hypercare
  Entry: Phase 3 gate passed
  Exit:  30-day stability window complete
  Gate:  Sponsor program closure sign-off
```

### Evidence Architecture

```
docs/06-evidence/
  ├── vendor/          ← All vendor communications archived here
  ├── emails/          ← Email threads linked, not summarized
  ├── configs/         ← Configuration snapshots with timestamps
  └── screenshots/     ← Visual proof with descriptive filenames

Every claim in the program must link to evidence.
Assertions without evidence links are rejected.
```

### Decision Severity Model

| Severity | Examples | Approver |
|----------|----------|----------|
| P1 | Scope change, cutover date, rollback invocation | Sponsor |
| P2 | Wave ordering, vendor selection, tech choice | PM + Tech Lead |
| P3 | Documentation format, template selection | PM |
| P4 | Minor wording, typo fixes | Author |

---

## Trade-offs

| Decision | Trade-off Made | Rationale |
|----------|----------------|----------|
| Git as single source of truth | Less accessible for non-technical stakeholders | Immutability + traceability outweighs access friction |
| Explicit AI authority limits | Slower AI-assisted workflows | Governance integrity — AI drafts, humans approve |
| Evidence-required for all claims | Documentation overhead | Downstream audit protection and rollback clarity |
| Phase gates before execution | Slower initial progress | Prevents executing against an invalid design |

---

## Outcome

- **Complete governance framework** before first migration workload touched
- **Role-authority model** prevents AI from overstepping into decision-making
- **Evidence architecture** ensures every decision is defensible in post-migration audit
- **Decision severity model** routes approvals to right level without bottlenecking on sponsor

### What I Would Do Differently

1. **Earlier stakeholder mapping** — RACI created before technical planning, not during
2. **Automated evidence ingestion** — scripts to pull configs into evidence/ on a schedule
3. **Decision log as living document** — real-time logging during execution, not reconstructed after

---

## Operational Lessons

1. **Governance frameworks are most valuable before they're needed** — build them during planning, not during a cutover crisis
2. **"Nothing executed unless approved" only works if approval is fast** — slow approval creates pressure to bypass; design approval flows to be frictionless for low-severity decisions
3. **AI as documentation engine, not decision engine** — Claude drafts runbooks and analysis; humans retain approval authority
4. **Configuration snapshots are the most valuable evidence** — more than emails, more than meeting notes, timestamped configs are the ground truth
5. **Phase gates must have explicit exit criteria** — "phase complete" means nothing; "all 23 systems inventoried and validated" means everything
