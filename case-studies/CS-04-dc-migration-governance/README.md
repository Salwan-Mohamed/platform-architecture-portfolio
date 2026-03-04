# Case Study 04: Governance-First Datacenter Migration Platform

**Type:** Migration Governance Design + Program Architecture  
**Scale:** Full datacenter migration program  
**Duration:** 2026 (design complete, execution phase starting)  
**Role:** Platform Architect + Program Governance Designer  
**Key Repos:** `dc-migration-platform`

---

## The Problem

Datacenter migrations have a predictable failure pattern. It's not usually a technical failure. The servers move. The network reconfigures. The applications come back up. The failure is organizational:

- **"We discussed this in the meeting"** — but the meeting had no minutes, the decision was verbal, and three months later nobody agrees on what was decided
- **"I thought you approved that"** — but approval was an email with "looks good" at 11pm, not a formal sign-off with scope and risk acknowledgment
- **"We'll document it after"** — but "after" never comes, and the evidence for what was done and why is scattered across Slack threads and email chains
- **"The AI can handle the documentation"** — but AI-generated documentation without human review and approval is decoration, not governance

The problem I was solving: how do you build a migration program where governance is the default, not the exception?

---

## The Environment

| Dimension | Details |
|-----------|--------|
| Program type | Full datacenter migration |
| Governance model | Human-in-loop always; AI assists, humans approve |
| Documentation standard | Evidence-based; every claim links to evidence |
| Approval model | Named approvers, dated, scoped, severity-rated |
| Change control | All changes to main require PM approval via PR |
| AI role | Claude generates drafts; humans review before merge |

---

## The Constraints

1. **University governance requirements** — Formal change management with audit trail required
2. **Distributed decision authority** — Some decisions require sponsor-level approval; others are PM authority
3. **AI-assisted workflow** — Claude Code is available for documentation generation, but AI cannot be an authority
4. **Evidence gap risk** — In complex migrations, evidence is often not collected until after incidents occur. Prevention requires structural forcing functions.
5. **Timeline pressure** — Migrations have business deadlines that create pressure to skip governance steps

---

## The Architecture: Governance as Repository Structure

The insight that drove the design: **governance can be embedded in repository structure**. If the structure forces you to produce governance artifacts before execution artifacts, governance becomes the path of least resistance rather than an obstacle.

```
dc-migration-platform/
  docs/
    00-executive/          ← Steering artifacts (written before execution starts)
    01-governance/         ← RACI, phase gates, decision severity, change control
                              ALL populated before any technical work begins
    02-current-state/      ← As-is discovery (evidence-based, timestamped)
    03-target-state/       ← To-be architecture (after current state is known)
    04-delta-plan/         ← Gap analysis, waves, cutover, rollback
    05-runbooks/           ← Operational procedures (written before execution)
    06-evidence/           ← Emails, configs, screenshots, vendor docs
```

The ordering is intentional and enforced:
- You cannot write target-state without current-state (governance constraint)
- You cannot write the delta plan without both states (logical constraint)
- You cannot execute without approved runbooks (process constraint)
- Every artifact links to evidence in `06-evidence/` (structural constraint)

---

## The AI Governance Model

This is the part of this case study that has the widest implications beyond DC migration.

The question isn't "should AI assist with DC migration documentation?" — of course it should. The question is **"what is the exact boundary between AI assistance and human authority?"**

| Action | Performed By | Authority Required |
|--------|-------------|-------------------|
| Draft document | Claude | None — drafts have no authority |
| Analyze current state | Claude Code | None — analysis is advisory |
| Identify gaps | Claude | None — gaps are flagged, not decided |
| Propose options with trade-offs | Claude | None — proposals require evaluation |
| Approve a decision | Human PM or Sponsor | Named authority, dated, scoped |
| Accept a risk | Human Sponsor | Sponsor authority only |
| Merge to main | Human PM | PR approval required |
| Close a phase gate | Human Sponsor | Explicit gate sign-off |

**The invariant: AI amplifies human capability. AI does not replace human authority.**

This model is Principal-level thinking because it solves the general problem ("how do you govern AI-assisted infrastructure work") not just the specific problem ("how do you document a DC migration").

---

## Phase Gate Model

```
Phase 1: Discovery & Assessment
  └─ Gate: Current state documented → Sponsor sign-off required
  
 Phase 2: Target Architecture  
  └─ Gate: To-be approved → Sponsor + Architecture review required

Phase 3: Migration Planning
  └─ Gate: Wave plan approved, rollback tested → PM + Ops sign-off

Phase 4: Execution (Wave by Wave)
  └─ Gate per wave: Success criteria met → PM approval before next wave

Phase 5: Hypercare
  └─ Gate: Stability confirmed over 30 days → Sponsor program close

No gate auto-completes. Time elapsed alone does not pass a gate.
```

---

## The Trade-offs

| What I gave up | What I gained |
|----------------|---------------|
| Speed (documentation takes time) | Legal and audit protection |
| Flexibility to skip steps under pressure | Organizational trust from stakeholders |
| AI-generated final documents | Human-reviewed accurate governance artifacts |
| Implicit approval culture | Explicit accountability culture |

---

## Outcome

- **Repository structure** created and fully templated — governance artifacts precede technical artifacts by design
- **AI authority model** explicitly defined — Claude drafts, humans approve, no exceptions
- **Phase gate framework** defined — no migration wave executes without gate sign-off
- **Evidence requirements** documented — every claim links to traceable evidence
- **Execution phase** starting — discovery in progress

---

## Operational Lessons

### Lesson 1: The decision log is more valuable than the technical design
Technical designs describe what you planned. The decision log describes what you actually decided and why. These diverge constantly during execution. Post-migration, the decision log is what allows you to answer: "Why is this configured this way?"

### Lesson 2: AI governance must be explicit before the first AI-generated artifact
Once AI generates a document that gets merged without clear authority boundaries, the governance model is broken. Defining the model before the first Claude interaction prevented ambiguity about which artifacts were authoritative.

### Lesson 3: Structure enforces governance more reliably than process documentation
A process document that says "get approval before proceeding" will be skipped under pressure. A repository structure where the approval file must be populated before the execution file can be created enforces the same requirement structurally.

---

*This case study demonstrates the design of a governance system that makes the right thing the default behavior — the hallmark of Principal-level platform thinking.*
