# LinkedIn Content — Salwan Mohamed
## Ready-to-Use Professional Positioning

---

## Headline Options

**Option 1 (Technical / Architect-focused):**
> Platform Engineer | Designing Governance-First Infrastructure Platforms | Nexus Platform | Multi-Site University Architecture | GitOps + Zero-Trust

**Option 2 (Principal-track / Strategic):**
> Senior Platform Engineer → Principal | Building Infrastructure Platforms Organizations Can Trust at Scale | 183 ADRs | Sinai University | GitOps-First

**Option 3 (Concise / Punchy):**
> Platform Architect | I build platforms that coordinate organizations, not just run workloads | GitOps | Zero-Trust | Multi-Site

---

## About Section

> Most infrastructure problems aren't technical. They're organizational.
>
> I build platform engineering systems for multi-site organizations — the kind that have three campuses, mixed technical maturity, small operations teams, and real users who can't afford downtime. The kind where "let's discuss this in the meeting" isn't a governance model.
>
> My work centers on one insight: a platform is an organizational coordination system implemented with GitOps. What you build with Kubernetes, ArgoCD, and FortiGate matters. But what makes the platform trustworthy over time — across team changes, vendor changes, and organization changes — is governance architecture.
>
> At Sinai University, I designed and built the Nexus Platform: 183 vendor-agnostic Architecture Decision Records defining platform doctrine, implemented with 14 environment-specific decisions covering RKE2, Cilium, ArgoCD, FortiGate 1101E, Keycloak, and VMware vSphere across 3 campuses.
>
> I also designed the Platform Operating System — a signal-driven decision governance framework where AI agents assist but humans decide, where every decision references observable signals, and where "silence is not consent."
>
> If you're building a platform that needs to be trustworthy in 2 years, not just working today, I'd be interested in the conversation.
>
> **Stack:** RKE2 | Cilium | ArgoCD | FortiGate | Keycloak | Terraform | Ansible/AWX | Prometheus/Grafana/Loki | Cluster API (CAPV)  
> **Domains:** Platform Engineering | Network Architecture | GitOps | Zero-Trust Security | Governance Design

---

## Featured Projects

### 1. Nexus Platform Engineering
> *Vendor-agnostic platform governance core — 183 Architecture Decision Records covering infrastructure, security, observability, pipelines, scaling, and organizational coordination. Not a Kubernetes platform. An organizational coordination system implemented with GitOps.*

### 2. Nexus Platform — Sinai University Implementation
> *Enterprise Platform for a 3-campus university — RKE2, Cilium, ArgoCD, FortiGate 1101E, Keycloak, vSphere. 14 implementation ADRs, each traceable to the vendor-agnostic core. Phase A active: Infrastructure Control operational.*

### 3. University Network Architecture
> *Multi-site network design documentation system — full-mesh IPsec VPN across 3 FortiGate edge firewalls, 67 firewall policies, 34 interfaces documented. Design-first methodology: architecture intent lives separately from operational state.*

### 4. Platform Operating System
> *Signal-driven decision governance framework — authority contracts, fitness functions, phase gates. The system that ensures platform decisions remain valid over time. AI assists; humans decide.*

---

## Post Ideas

### Post 1: Architecture Insight
> **"The most dangerous state for infrastructure: operational but undocumented."**
>
> I ran a full-mesh VPN across 3 campus sites. When I finally documented the Kantra-site FortiGate configuration (34 interfaces, 67 policies, 5 VPN tunnels), I found it: 1,135 IPsec Phase 2 negotiation failures on the Cairo-S2S tunnel. The tunnel looked like it was working. IKE Phase 1 was up. But zero traffic was passing.
>
> We found it because we documented the baseline before trying to optimize.
>
> Documentation is not overhead. It's the instrument you use to find what you don't know you're missing.

### Post 2: Governance Insight
> **"The difference between a platform and a platform product: the ADR."**
>
> Anyone can stand up Kubernetes. I've done it many times.
>
> What makes a Kubernetes cluster a *platform* is the ADR that says: why RKE2 over kubeadm? What was the constraint? What was given up? What would need to change to reconsider this decision?
>
> I wrote 197 of them. Not because I had to. Because the next engineer who touches this environment deserves to understand why things are the way they are.
>
> The ADR is how decisions survive the people who made them.

### Post 3: Architecture Philosophy
> **"Full-mesh VPN is not a performance optimization. It's a resilience architecture."**
>
> When I designed the Sinai University network, the default answer was hub-spoke: all inter-campus traffic via the datacenter.
>
> I rejected it.
>
> Hub-spoke means: if the datacenter is in maintenance, Kantra can't reach Arish. If the datacenter has an unplanned outage, three campuses lose connectivity to each other.
>
> Full-mesh means: branches communicate directly. Losing the datacenter is a datacenter problem. It's not a campus communication problem.
>
> Network topology is organizational design expressed in packets.

---

## Skill Endorsements to Highlight

Platform Engineering | Kubernetes | GitOps | Network Architecture | Infrastructure Architecture | Ansible | Terraform | ArgoCD | Cilium | FortiGate | VMware vSphere | Architecture Decision Records | Technical Documentation | Zero-Trust Security
