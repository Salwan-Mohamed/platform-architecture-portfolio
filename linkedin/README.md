# LinkedIn Content — Salwan Mohamed

> Ready-to-paste LinkedIn content. Each section includes options. Human owns final voice — these are drafts.

---

## Headline Options

**Option A (Technical):**
```
Platform Engineer | Building Governance-First Infrastructure Platforms | GitOps · Kubernetes · Multi-Site Architecture
```

**Option B (Strategic):**
```
Principal Platform Engineer | Nexus Platform Architect | Turning Infrastructure Chaos into Coordinated Systems
```

**Option C (Outcome-Focused):**
```
Platform Engineer Building University Infrastructure at Scale | 183 ADRs · RKE2 · FortiGate · Full-Mesh VPN
```

**Recommended:** Option B for Principal-track positioning. Option A for technical recruiter reach.

---

## About Section

```
I build platforms that organizations can trust — not just infrastructure that works until it doesn't.

My work is at the intersection of three things: architecture thinking, engineering governance, and organizational coordination. The platforms I build are defined by what they coordinate, not what they're built on.

Over the past two years, I've designed and built the Nexus Platform System from first principles:

→ 183 Architecture Decision Records documenting every significant technical choice, vendor selection, and design trade-off
→ A multi-site platform for Sinai University: 3 campuses, 5,800 users, FortiGate full-mesh VPN, RKE2 Kubernetes, GitOps-first operations
→ A Platform Operating System — a signal-driven governance framework that ensures every architectural decision references observable evidence, not opinion
→ A Datacenter Migration Platform with human-in-loop governance, explicit AI authority boundaries, and evidence-based phase gates

What separates my work from "infrastructure engineering" is the governance layer:

• Every decision has a named owner and documented rationale
• AI tooling is bounded by authority contracts — advisors, not decision-makers
• Platform maturity follows phase gates — advancement is earned, not assumed
• Infrastructure is under version control before features are added

I'm on the Senior → Principal Platform Engineer track. The portfolio is public:
🔗 github.com/Salwan-Mohamed/platform-architecture-portfolio

Open to conversations about principal platform engineering, platform architecture for multi-site organizations, and governance-first infrastructure design.
```

---

## Project Descriptions

### Nexus Platform Engineering Core
```
Built a vendor-agnostic platform governance system from first principles.

The challenge: how do you separate what your platform believes from what your platform runs on — so that vendor changes don't require governance rebuilds?

Solution: a Core/Product separation model. The Core defines 183 Architecture Decision Records covering governance, observability (8 axes), security, network, pipeline, and organizational architecture — all vendor-agnostic. Product repositories implement these contracts with specific tool choices.

The Core reached v14.0 in STEWARDSHIP MODE: architecturally complete. Now deployed as the governance foundation for Sinai University's infrastructure platform.

Tech: Architecture Decision Records · GitOps governance · Phase-based maturity · Vendor-neutral design
```

### Sinai University Platform
```
Designed and built a full-stack infrastructure platform for a 3-site, 5,800-user Egyptian university — from zero automation to GitOps-first operations.

Starting state: manual CLI changes, no audit trail, no version control, a 40-minute unexplained outage with no post-mortem possible.

Built: Full-mesh IPsec VPN with BGP routing between 3 sites. RKE2 Kubernetes with Cilium network policy. FortiGate 1101E security boundaries with centralized FortiManager policy. Keycloak identity brokering over Active Directory. 8-axis observability with Prometheus/Grafana/Loki. All of it managed as code via ArgoCD + Ansible AWX.

Outcome: 100% infrastructure under version control. Every change traceable. Zero unauthorized production changes since GitOps adoption. Phase A complete; Phase B in progress.

Tech: RKE2 · ArgoCD · Cilium · FortiGate · VMware vSphere · Keycloak · Ansible/AWX · Terraform
```

### Platform Operating System
```
Designed a signal-driven governance framework for architectural decision-making.

The problem: as the platform scaled, decisions were being made based on opinion and urgency rather than observable evidence. AI tooling was creating governance theater — generating approvals without meaningful human oversight.

Built: A decision governance system where every architectural choice must reference a signal from the canonical Signals Registry. AI agents are bounded by explicit authority contracts: they draft, they analyze, they execute — but they don't approve. The Human Owner is the sole decision authority.

Currently in Phase 10: Signal Activation & Baseline Establishment. Observe before enforce. 30-day observation window before any enforcement is activated.

Tech: Signal-driven architecture · Authority contracts · MCP integrations · Claude Code governance boundaries
```

### DC Migration Platform
```
Designed a governance-first datacenter migration platform — because DC migrations fail from governance failures, not technical ones.

The design challenge: incorporate AI tooling (Claude Code) in a migration program without creating governance theater — where AI-generated documents get treated as approved artifacts.

Solution: explicit role architecture. Claude drafts. Humans approve. No artifact is final until a named human with the appropriate authority has signed off. Decision severity classification (P1/P2/P3) ensures the right approver for each decision type. Evidence lives in the repository — no separate audit assembly at program close.

Outcome: zero unauthorized production changes during migration planning phase. Every P1/P2 decision traceable to a named human owner.

Tech: Governance-first design · Evidence management · AI authority boundaries · Phase-gate architecture
```

---

## Post Ideas (Architecture Insights)

### Post 1: The Core/Product Separation
```
Most platform teams build their governance around their tool choices.

That means when the tool changes — and it always does — you rebuild the governance.

I took a different approach: separate what your platform BELIEVES from what your platform RUNS ON.

The Nexus Platform Core defines 183 architectural decisions that are vendor-agnostic. Tool names never appear. What appears instead: capability contracts, authority models, phase definitions, and governance principles.

The Sinai University Product implements those contracts. FortiGate satisfies the "Firewall Control Plane" contract. RKE2 satisfies the "Kubernetes Control Plane" contract. Keycloak satisfies the "Identity Broker" contract.

If we replace FortiGate tomorrow, we update the Product. The Core doesn't move.

That's not a minor organizational decision. That's the difference between a platform team and an infrastructure team.
```

### Post 2: Phase A Before Phase D
```
Every platform project I've seen has the same temptation: build the developer portal before the infrastructure is under version control.

Phase D features (self-service, golden paths, internal developer portal) before Phase A stability (everything in Git, audit trail, monitoring) is how you build a showroom on a foundation you don't trust.

At Sinai University, leadership wanted the portal in month 3.

We delivered the portal in month 9 — after Phase A was complete.

The conversation was hard. The outcome was worth it: when the portal launched, it ran on infrastructure that was fully observable, fully auditable, and operated by a team that trusted their own systems.

Advancement to the next phase is optional. It is not assumed. That distinction is the whole model.
```

### Post 3: AI Governance Theater
```
The risk with AI in platform engineering isn't that it will make bad decisions.

It's that it will make the APPEARANCE of good governance without the substance.

AI can generate a RACI in 2 minutes. It can draft an ADR in 5. It can produce a phase gate document that looks exactly like the real thing.

None of that is governance. Governance is when a named human with accountability reads it, challenges it, and signs their name to it.

In the DC Migration Platform, I defined Claude's role explicitly before the program started:
- RECOMMEND only
- All outputs require human review before becoming authoritative
- No artifact is final until a named human signs off

That's not a limitation on AI. That's a clarity of purpose.

Intelligence without authority is noise.
```

---

## Featured Section Recommendations

Pin these to LinkedIn Featured:

1. **platform-architecture-portfolio** — "Principal Platform Engineer Architecture Portfolio"
2. **nexus-platform-engineering** — "183-ADR Vendor-Agnostic Platform Governance Core"
3. A Medium article about one of the case studies (when written)

---

## Career Track Statement

**For interviews / profile positioning:**
```
I am on the Senior → Principal Platform Engineer track.

The progression is evidenced by three things:

1. ARCHITECTURE THINKING: 183 ADRs documenting every significant design decision, trade-off, and constraint across the Nexus Platform system. Each ADR follows the structure: Context → Constraints → Decision → Alternatives → Trade-offs → Consequences → Evidence.

2. GOVERNANCE DESIGN: A signal-driven Platform Operating System that defines how decisions are made, validated, and enforced — including explicit authority boundaries for AI tooling.

3. OPERATIONAL OUTCOMES: Phase A complete at Sinai University — 3 sites, 5,800 users, full-mesh VPN, GitOps-first operations, zero unauthorized changes, 100% infrastructure under version control.

The portfolio is at: github.com/Salwan-Mohamed/platform-architecture-portfolio
```
