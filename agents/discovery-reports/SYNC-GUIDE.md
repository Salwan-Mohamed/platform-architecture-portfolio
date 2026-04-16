# Portfolio Sync Guide

> How to keep the portfolio current as source repos evolve

---

## The Two-Track IMPL System

The `nexus-platform-product-sinai-university` repo has two IMPL ADR tracks:

| Track | Path | Phase | Count | Portfolio Status |
|-------|------|-------|-------|------------------|
| Phase A | `docs/01-decisions/adr/impl-000X` | A | 20 | ✅ ADR-013–016 added 2026-04-17 |
| Phase B | `docs/adr/IMPL-00XX` | B | 11 | ✅ ADR-009–012 added 2026-04-16 |

---

## Complete ADR Map: Source → Portfolio

### Phase A Track (impl-0001 to impl-0020)

| Source IMPL | Topic | Portfolio ADR | Status |
|-------------|-------|---------------|--------|
| impl-0001 | FortiGate firewall | ADR-004 | ✅ Exists (predates sync) |
| impl-0002 | ArgoCD GitOps | ADR-013 | ✅ Added 2026-04-17 |
| impl-0003 | RKE2 Kubernetes | ADR-003 | ✅ Exists (predates sync) |
| impl-0004 | vSphere virtualization | ADR-014 | ✅ Added 2026-04-17 |
| impl-0005 | Active Directory identity | — | Covered by ADR-009 |
| impl-0006 | Keycloak identity broker | ADR-009 | ✅ Added 2026-04-16 |
| impl-0007 | Prometheus metrics | ADR-015 | ✅ Added 2026-04-17 |
| impl-0008 | Grafana visualization | ADR-015 | ✅ Added 2026-04-17 |
| impl-0009 | Loki log aggregation | ADR-015 | ✅ Added 2026-04-17 |
| impl-0010 | Cilium network policy | ADR-010 | ✅ Added 2026-04-16 |
| impl-0011 | Terraform provisioning | ADR-016 | ✅ Added 2026-04-17 |
| impl-0012 | AWX runbook automation | ADR-016 | ✅ Added 2026-04-17 |
| impl-0013 | Tekton CI pipelines | ADR-016 | ✅ Added 2026-04-17 |
| impl-0014 | CAPV cluster lifecycle | ADR-014 | ✅ Added 2026-04-17 |
| impl-0015 | Sealed Secrets | ADR-016 | ✅ Added 2026-04-17 |
| impl-0016 | Robusta alert enrichment | ADR-015 | ✅ Added 2026-04-17 |
| impl-0017 | Pipeline separation (CI/CD) | ADR-013 | ✅ Added 2026-04-17 |
| impl-0018 | Credential lifecycle | ADR-016 | ✅ Added 2026-04-17 |
| impl-0019 | Infrastructure TDD | — | ⏳ Candidate: ADR-017 |
| impl-0020 | Static analysis | — | ⏳ Candidate: ADR-017 |

### Phase B Track (IMPL-0021 to IMPL-0031)

| Source IMPL | Topic | Portfolio ADR | Status |
|-------------|-------|---------------|--------|
| IMPL-0021 | DNS domain strategy | — | ⏳ Candidate: ADR-018 |
| IMPL-0022 | Storage class strategy (Longhorn) | — | ⏳ Candidate: ADR-018 |
| IMPL-0023 | OIDC authentication | ADR-009 | ✅ Added 2026-04-16 |
| IMPL-0024 | Group-based authorization | ADR-009 | ✅ Added 2026-04-16 |
| IMPL-0025 | SLO operating model | ADR-011 | ✅ Added 2026-04-16 |
| IMPL-0026 | Alerting strategy | ADR-011 | ✅ Added 2026-04-16 |
| IMPL-0027 | Phase transition criteria | ADR-012 | ✅ Added 2026-04-16 |
| IMPL-0028 | Infrastructure execution parameters | — | Covered by ADR-016 |
| IMPL-0029 | CNI / Cilium | ADR-010 | ✅ Added 2026-04-16 |
| IMPL-0030 | Execution tooling strategy | ADR-016 | ✅ Added 2026-04-17 |
| IMPL-0031 | North-south exposure model | — | ⏳ Candidate: ADR-018 |

---

## Remaining ADR Candidates

Three ADRs remain to be written:

| Portfolio ADR | Source IMPLs | Topic | When |
|---------------|--------------|-------|------|
| ADR-017 | impl-0019, impl-0020 | Infrastructure TDD + Static Analysis | Next sync |
| ADR-018 | IMPL-0021, IMPL-0022, IMPL-0031 | DNS Strategy + Storage + North-South Exposure | Next sync |

---

## Claude Code Agent Output

The Claude Code agent (2026-04-16) created 9 files in the TaskOS vault proposing:
- Cilium ADR (already in portfolio as ADR-010)
- ArgoCD + Flux GitOps ADR (covered by ADR-013)
- SPIFFE/SPIRE workload identity ADR (forward-looking — not yet an IMPL)
- Entra ID as unified IdP (forward-looking — not yet an IMPL; current decision is Keycloak)
- Hubble observability ADR (covered by ADR-010 and ADR-015)
- Case study CS-01 update (covered by CS-01 update on 2026-04-16)
- Two Mermaid diagrams (identity architecture, Cilium/Hubble observability)

**The two diagrams** are the only unsynced artifacts with net-new value. To sync:
```
taskos_read_file objects/OBJ-20260416-diagram-identity-architecture-sinai.md
taskos_read_file objects/OBJ-20260416-diagram-cilium-hubble-observability-sinai.md
```
Then push both to `system-diagrams/` in the portfolio.

**SPIFFE/SPIRE and Entra ID** — treat as ADR candidates pending human review. They are architectural recommendations, not current decisions. Do not add to the portfolio ADR library until a decision is made and documented in the source IMPL repo.

---

## How to Run the Next Sync

```bash
cd platform-architecture-portfolio
claude
```

```
# Step 1: Check for new IMPLs in source repo
Discovery Agent: analyze nexus-platform-product-sinai-university

# Step 2: Write remaining ADRs
Architect Agent: write ADR-017 from impl-0019 infrastructure TDD and impl-0020 static analysis
Architect Agent: write ADR-018 from IMPL-0021 DNS strategy, IMPL-0022 storage, IMPL-0031 north-south exposure

# Step 3: Sync the two diagrams from TaskOS vault
Diagram Agent: read OBJ-20260416-diagram-identity-architecture-sinai from taskos vault and commit to system-diagrams/
Diagram Agent: read OBJ-20260416-diagram-cilium-hubble-observability-sinai from taskos vault and commit to system-diagrams/

# Step 4: Update CS-01 with execution stack and observability details
Storyteller Agent: update CS-01 with ADR-013 through ADR-016 outcomes

# Step 5: LinkedIn posts
Positioning Agent: write LinkedIn post from ADR-016 infrastructure execution stack
Positioning Agent: write LinkedIn post from ADR-015 four-component observability stack
```
