# AIOps Agentic Reference Architecture — AMS, AD, IMS

---
**Type:** Reference
**Version:** 1.0
**Status:** Locked
**Date:** 2026-06-03
**Role:** Pattern library for agentic AIOps systems — a menu of architectural options with documented trade-offs, not a template to copy
**Inputs:** N/A (read-only reference)
**Outputs:** N/A

---

## How to use this document

**Who reads this:** This document is the primary input to the AIOps architecture design session (`2a-aiops-prompt-architecture.md`). The architect prompt instructs the LLM to read it in full before designing. It may also be attached to any session where architectural trade-offs need to be evaluated.

**What this document is:** A pattern library derived from production-grade AIOps programmes across three service lines (AMS, AD, IMS). Each concern is documented once, with the decisions, trade-offs, and rationale that apply across all three. §15 consolidates the service-line-specific application of each pattern.

**What this document is not:** It is not a template, a blueprint, or a prescription. No pattern should be included in a client design without a supporting F-## finding or explicit client constraint. The architect selects from this menu based on what the client's data and context warrant — no more, no less.

**How to read it selectively:** The document is structured by architectural concern (§1–§14), not by service line. Start with §1 (Architecture overview) to understand the principles and decisions. Then read only the concerns relevant to the engagement:
- Concern areas that almost always apply: §2 (Compute), §3 (Data), §6 (Agent orchestration), §9 (Observability), §11 (Security), §12 (Governance)
- Concern areas to read when relevant: §4 (Knowledge), §5 (ML pipelines), §7 (Agentic memory), §8 (Integration), §10 (Reliability), §13 (Quality/AgentOps), §14 (FinOps)
- Service-line specifics: §15

**Relationship to the 14-UC taxonomy and F-## findings:** The canonical 14 use cases (UC-01 through UC-14) are the bridge between this reference and any client engagement. Every architectural pattern in this document should be traceable to one or more use cases. When the architect proposes a component, they should be able to name the UC it serves and the F-## finding that motivated it.

**Scope:** AMS, AD, and IMS service lines. All cloud-side components run on AWS. Kiro SDD agents run on developer laptops (AD only) — the only off-AWS surface. §19 covers alternatives for Azure or platform-independent deployments.

**Current status:** Architecture locked. All 40 open questions triaged and resolved (see §16). Ready for component-level decomposition (§17).

---

## Table of contents

1. [Architecture overview](#1-architecture-overview)
2. [Compute & runtime](#2-compute--runtime)
3. [Data architecture](#3-data-architecture)
4. [Knowledge architecture](#4-knowledge-architecture)
5. [ML & data pipelines](#5-ml--data-pipelines)
6. [Agent orchestration](#6-agent-orchestration)
7. [Agentic memory](#7-agentic-memory)
8. [Integration plane](#8-integration-plane)
9. [Observability](#9-observability)
10. [Reliability & resilience](#10-reliability--resilience)
11. [Security & threat response](#11-security--threat-response)
12. [Governance](#12-governance)
13. [Quality & evaluation (AgentOps)](#13-quality--evaluation-agentops)
14. [FinOps & cost](#14-finops--cost)
15. [Service-line application](#15-service-line-application)
16. [Resolved decisions log](#16-resolved-decisions-log)
17. [Next step — component decomposition](#17-next-step--component-decomposition)
18. [Per-deployment trade-offs to re-surface](#18-per-deployment-trade-offs-to-re-surface)
19. [Cloud strategy: AWS, Azure, platform-independent](#19-cloud-strategy-aws-azure-platform-independent)
20. [High-level task list for delivery estimation](#20-high-level-task-list-for-delivery-estimation)
21. [License requirements](#21-license-requirements)

Appendices:
- [Appendix A — Glossary](#appendix-a--glossary)
- [Appendix B — Segment vs. Domain](#appendix-b--segment-vs-domain-keep-distinct)

---

## 1. Architecture overview

### 1.1 Three-layer view

The architecture is organised in three layers. Every component in this document belongs to exactly one layer.

| Layer | Purpose | Concerns in this layer |
|---|---|---|
| **Foundation** | Provides data, knowledge, and substrate signals agents reason over | §3 Data, §4 Knowledge, §5 ML & data pipelines |
| **Orchestration** | Runs the agents, brokers their tool and peer calls, manages their memory | §2 Compute & runtime, §6 Agent orchestration, §7 Agentic memory, §8 Integration plane |
| **Trust** | Watches, governs, evaluates, and pays for everything above | §9 Observability, §10 Reliability, §11 Security, §12 Governance, §13 Quality, §14 FinOps |

Service lines (§15) and use cases (§15) consume all three layers.

### 1.2 Principles

| ID | Principle |
|---|---|
| **P1** | One reference, three deployments. AMS, AD, IMS share the same architectural concerns; differences are deployment variations, not architecture variations. |
| **P2** | Build once at programme level. Shared services (KL, Governance Sidecar, MCP plane, AgentOps, Observability, Audit) exist exactly once and serve all three service lines. |
| **P3** | **Cloud strategy is a per-engagement choice.** This reference architecture supports three strategies: AWS-native, Azure-native, and platform-independent (Kubernetes + OSS). The programme default is AWS-native (locked as D2) with Kiro on developer laptops as the only off-cloud surface. Alternative strategies and the mapping of every component across all three are in §19. |
| **P4** | Managed-first for plumbing, custom for differentiation. Adopt the chosen cloud's managed primitives (per §19: AgentCore on AWS, Foundry Agent Service on Azure, OSS Kubernetes substrate platform-independent) for the agent substrate; build custom only where service-line logic differentiates. |
| **P5** | MCP for tools, A2A for agents. MCP is the universal protocol for agent-to-tool. A2A is the protocol for agent-to-agent across runtime or organisational boundaries. |
| **P6** | Standards over vendor-specific schemas. OpenTelemetry GenAI Semantic Conventions for AI telemetry; OpenLineage for data lineage; FOCUS for billing data; OWASP ASI 2026 for agentic threats. Tooling is chosen on the basis of standards compatibility. |
| **P7** | Evaluation-first lifecycle. Every agent goes through the Agent Development Lifecycle (§12.2) with eval and red-team gates before deployment. |
| **P8** | Fail safe, never bypass governance. When a managed dependency is unavailable, agents degrade or stop — they do not bypass policy enforcement. |
| **P9** | Identity propagates end-to-end. Developer or service identity flows from invocation through orchestrator, LLM Gateway, MCP Gateway, and into the receiving tool. No anonymous agent actions. |
| **P10** | Trace context propagates end-to-end. W3C Trace Context flows through orchestrator, LLM Gateway, MCP plane, and A2A boundaries. Disconnected traces are a defect. |

### 1.3 Programme-level decisions

The decisions below capture the **locked default** position for this programme (AWS-native, per P3/D2). For deployments where AWS is not the right fit, **§19 maps every cloud-coupled decision (D2, D5, D8, D12, D13, D15, D16, D17, D18) to its Azure-native and platform-independent equivalents**. The non-cloud-coupled decisions (D1, D3, D4, D6, D7, D9, D10, D11, D14, D19, D20, D21, D22, D23) hold across all three strategies.

| ID | Decision | Rationale |
|---|---|---|
| **D1** | Three parallel reference deployments (AMS, AD, IMS), one architecture | Service lines have different shapes (reactive ticketing, developer-facing, broad infra); a unified RA forces bad compromises. The concern-based architecture is shared; deployments vary. |
| **D2** | AWS as the locked default cloud for this programme; Kiro on developer laptops the only off-cloud surface. Azure-native and platform-independent strategies are documented in §19 for deployments where AWS is not the right fit (existing Azure footprint, sovereignty, multi-cloud mandate). | Programme constraint at locked default; §19 surfaces the alternatives. |
| **D3** | Knowledge Layer, Governance Sidecar, MCP plane, AgentOps, Observability, Audit Log built once at programme level | P2. Avoids duplication; enforces consistency. |
| **D4** | Per-service-line LLM Gateway, agent runtime, and MCP servers | Cost allocation, governance, data-residency profiles differ per line. The Gateway and runtime are the same architecture (§2, §8); only configurations vary. |
| **D5** | Hybrid Bedrock AgentCore + LangGraph | AgentCore Runtime, Gateway, Identity, Memory, Observability are managed (the plumbing); LangGraph is the orchestration framework (the brain). See §2.2 and §6.1. |
| **D6** | OpenTelemetry GenAI Semantic Conventions are mandatory for agent instrumentation | P6. Prevents vendor lock-in across AI observability tools. |
| **D7** | A2A protocol for cross-service-line agent flows | P5. MCP is agent-to-tool only; A2A is agent-to-agent across runtimes. |
| **D8** | Bedrock as the model plane; fallback chains within Bedrock model families | P4. Single model plane simplifies governance, audit, identity, billing. |
| **D9** | OPA at the Governance Sidecar; policy-as-code uniform across all three lines | P8. One policy language; one audit format. |
| **D10** | Maturity-staged capability sequencing (Foundation → Signal → Predict → Act → Govern → Autonomy → Learn) | Service-line pillars (§15) numbered by this sequence. |
| **D11** | Continuous Learning capability per service line is the ADLC Runtime Optimization loop | §12.2. Not a use-case home; consumed by every other capability. |
| **D12** | GitHub Enterprise on AWS for code hosting; GitHub Actions on AWS runners for CI/CD | Mature ecosystem; CodeCommit deprecated 2025-26; satisfies data residency. CodeCatalyst remains optional via MCP. |
| **D13** | Hybrid retrieval store: pgvector on RDS PostgreSQL + OpenSearch BM25 | pgvector co-locates with transactional data (RDS), avoiding network hop; OpenSearch BM25 wins on keyword-heavy queries (CMDB lookups, CI names). Two stores accepted for retrieval quality. |
| **D14** | Knowledge graph deferred to Phase 3, only for AMS-01 / IMS-01 if hybrid retrieval hits quality ceilings | KG extraction is 3–5× LLM call cost (§4.4); most enterprise RAG succeeds without it. Add only when proven necessary. |
| **D15** | Reranker: Cohere Rerank on Bedrock | P4 managed-first; D8 single model plane; cost-competitive at expected volumes. Self-host only above ~10M reranks/month. |
| **D16** | AI observability + AgentOps shape: Langfuse (self-hosted) for prompt registry + agent traces; AgentCore Observability for OTel collection; Bedrock Evaluations for batch evals | Each tool does what it does best; avoids tying to Datadog. Langfuse OSS keeps full ownership of prompt registry. |
| **D17** | Memory framework: AgentCore Memory (managed) | P4 managed-first; D5 hybrid AgentCore + LangGraph. Swap to Mem0/Letta only if benchmarks prove AgentCore materially weaker for our workloads. |
| **D18** | Data observability: AWS-native (Glue Data Quality + OpenLineage on Marquez + custom rules) | Open standards (OpenLineage), no commercial license, fits AWS-only (P3). More engineering effort upfront; team owns the stack. |
| **D19** | Developer identity in audit log: pseudonymisation with two-person re-identification | Compatible with most works-council frameworks; preserves forensic capability for IP/governance investigations. Final per-jurisdiction tuning during rollout. |
| **D20** | Offline tolerance: AD-DWP reads + suggestion generation work offline; write paths (including local git commits) require connectivity | Preserves P8 (governance never bypassed) and P9 (identity propagation). Local agent suggestions remain available; persistence requires governance gate. |
| **D21** | Constitutional SDD: built on Kiro steering files + ADLC decision logs | §12.4. Commercial constitutional-SDD tooling is immature in 2026; build-on-Kiro avoids vendor risk. Swap later is low-cost. |
| **D22** | Multi-region deferred to Phase 4+ | Phase 1–3 prioritises depth over geographic spread. Multi-AZ from Phase 1 (D8 reliability). Multi-region added when regulatory or business continuity demands. |
| **D23** | Kiro version pinning, coordinated quarterly upgrade | Reproducibility of AI behaviour; regression evals run before promoting new Kiro version. |

### 1.4 Notation

| Marker | Meaning |
|---|---|
| **[CONFIRMED]** | Validated against public sources |
| **[ASSUMED]** | Working assumption awaiting confirmation |
| **§N.M** | Internal cross-reference to a section of this document |

### 1.5 Six AIOps capability pillars (AMS & IMS lens)

For AMS and IMS — the two AIOps-flavoured service lines — the capability surface can be viewed as six pillars covering the full incident-and-operations lifecycle. This is a **complementary lens** to the maturity-staged sequencing in D10 (Foundation → Signal → Predict → Act → Govern → Autonomy → Learn). Both views describe the same capability set; the maturity view drives sequencing, the pillar view drives scoping conversations.

| # | Pillar | Capability | Where it lands |
|---|---|---|---|
| 01 | **Signal Intelligence** | Noise suppression, event correlation, persistence thresholds, semantic deduplication | AMS-02, IMS-02 (on AMS-IP / IMS-IP) |
| 02 | **Intelligent Triage** | Priority scoring, skill-based routing, CMDB-aware assignment | AMS-03, in IMS-02 |
| 03 | **Knowledge Management** | KB generation from work notes, RAG-powered resolver copilot, self-service deflection | AMS-01, IMS-01 (shared KL); resolver-copilot surfaces on AMS-IP / IMS-IP |
| 04 | **Predictive Intervention** | Failure prediction, pre-check scheduling, MTBF-driven preventive tasks | AMS-04, IMS-04 (on AMS-PIS / IMS-PIS) |
| 05 | **Agentic Assistance** | Multi-step reasoning, autonomous diagnosis, governed action execution | AMS-05, IMS-03, IMS-05a, IMS-05b, IMS-06, IMS-08 (on AMS-ARP / IMS-AOP / IMS-CPP / IMS-SPP) |
| 06 | **Continuous Learning** | Outcome feedback loop, model retraining, recurring-issue tracking | AMS-07, IMS-09 (shared AgentOps platform; D11) |

Pillars 01–02 are typically deliverable without ML and ship early (Phase 1–2). Pillars 03–05 require data maturity and governance approvals and ship in Phase 2–3. Pillar 06 begins once outcome data is flowing (Phase 3) and compounds over time.

This pillar lens does **not** apply to AD. AD is software-engineering work (specification, design, code, test, release) and uses the AD pillar set (AD-01..AD-07) as defined in §15.3.

### 1.6 Four-tier autonomy maturity model

The autonomy levels referenced throughout this document (L1–L5 in §11, §15) follow a four-tier maturity model. The fifth level is a programme-internal extension reserved for closed-loop self-improvement in proven, low-risk domains.

| Tier | Name | Agent behaviour | Human role |
|---|---|---|---|
| **L1 — Assisted** | Agent enriches tickets and surfaces relevant context | All decisions, all actions |
| **L2 — Advisory** | Agent recommends (triage, KB drafts, routing, remediation plans) | Approves before any action |
| **L3 — Execution** | Agent executes defined workflows within guardrails | Sets policy, has override; HITL on high-stakes |
| **L4 — Autonomous (bounded)** | Agent self-governs within a proven domain | Sets goals; accountability retained; trust-decay restricts on SLI drop |
| **L5 — Closed-loop autonomous** | Agent participates in its own improvement loop via ADLC Runtime Optimisation (D11) | Programme-level oversight |

**Progression rules:**

- Progression is **per-action-type and per-domain**, not per-ticket. A single ticket may contain an L3 action for one domain and an L2 action for another.
- Common progression criteria: sustained safety SLI above threshold over a defined window (e.g., 8 weeks), error rate below threshold, domain-specific stability metrics.
- **Trust-decay** is mandatory at L4+: autonomy is automatically restricted when SLI drops below threshold. The OPA gate (§12.3) enforces the restriction within minutes.
- Some clients deliberately cap at L2 (advisory only) for regulatory or cultural reasons. This is a configuration choice, not a deviation.
- D10 (maturity-staged sequencing) and this tier ladder are independent axes. Phase 4 (autonomy theme, §15.5) is when L4+ becomes available; whether a given pillar reaches L4 is a per-domain risk-appetite call.

---

## 2. Compute & runtime

### 2.1 Compute taxonomy

The programme runs five distinct kinds of compute, each with a defined role.

| Compute kind | Where it runs | Primary role | Examples |
|---|---|---|---|
| **Container compute (EKS)** | AWS EKS clusters | Long-running services, custom orchestration logic, data pipelines, model-serving where Bedrock doesn't fit | LangGraph orchestrators, ingestion services, ML serving |
| **Managed agent runtime (AgentCore Runtime)** | AWS managed | Hosts agents with session management, scaling, identity propagation | All agentic platforms (AMS-ARP, IMS-AOP, IMS-CPP, IMS-SPP, AD-ACAP, AMS-IP/IMS-IP advisory agents) |
| **Serverless event compute (Lambda, Step Functions)** | AWS managed | Scheduled jobs, approval workflows, light glue logic, MCP egress shims | Approval workflows, FOCUS billing ingestion, scheduled audits |
| **Time-series / ML compute (SageMaker)** | AWS managed | Predictive model training, registry, batch inference | Predictive Intelligence Systems (AMS-PIS, IMS-PIS) |
| **Developer-laptop compute (Kiro IDE)** | Developer laptop | Interactive AI-assisted development | AD developer workspace only |

### 2.2 AgentCore Runtime + LangGraph (the agentic platform substrate)

The default agent execution model across all three service lines:

- **AgentCore Runtime** hosts the agent — provides serverless scaling, session management, identity propagation, observability hooks.
- **LangGraph** is the orchestration framework loaded into the runtime — provides explicit state-machine modelling of agent reasoning, checkpointing for human-in-the-loop pauses, and deterministic edge routing.
- **Bedrock** is the model plane reached through the LLM Gateway (§8.3).

The split is deliberate: managed substrate for the plumbing (which doesn't differentiate), open-source framework for the orchestration logic (which does).

### 2.3 Runtime requirements

Every runtime hosting agents must provide:

| Requirement | Mechanism |
|---|---|
| Session lifecycle (start, persist, resume, end) | AgentCore Runtime |
| Identity propagation (user → agent → tool) | AgentCore Identity (§8.4) |
| Trace context propagation (P10) | OTel auto-instrumentation + AgentCore Observability headers |
| Checkpointing for HITL pause/resume | LangGraph checkpointer (§6.5) |
| Memory access (working through procedural) | §7 |
| Tool access (Classes A, B, C) | §8 |
| Cost telemetry per invocation | LLM Gateway (§8.3) |
| Governance gating | OPA via Governance Sidecar (§12) |

### 2.4 When to use which compute

| Decision | Use |
|---|---|
| Multi-step reasoning agent with tools | AgentCore Runtime + LangGraph |
| Predictive model training and batch inference | SageMaker |
| High-throughput streaming ingestion | EKS (Kafka consumers, ingestion services) |
| Scheduled job, light glue, approval workflow | Lambda + Step Functions |
| Interactive AI in IDE | Kiro on developer laptop |
| Bulk data transformation | EKS (Spark on EKS) or Glue |

---

## 3. Data architecture

### 3.1 Premise

Agents fail at scale when data is fragmented, stale, or unowned. The data layer is treated as a first-class architectural concern — not as a passive store behind a query API — because every use case touches data, and unresolved data ambiguity is the single largest cause of stalled sizing exercises.

### 3.2 Data principles

| ID | Principle | Enforced by |
|---|---|---|
| **D-P1** | Data as a product | Data Contracts Registry (§3.4) |
| **D-P2** | Contract-backed access — every data product has schema, freshness SLA, access policy, lineage | Data Contracts Registry + KL query layer |
| **D-P3** | Entity-centric organisation — data products organised around business entities, not analytics tables | Schema design discipline |
| **D-P4** | Freshness as a published metric — agents that exceed staleness tolerance fail safe | §9.4 data observability |
| **D-P5** | End-to-end lineage — every data product traces back to sources and forward to consumers (including agents) | OpenLineage on Marquez (§9.4) |
| **D-P6** | Minimum viable data — agents receive the minimum payload needed for the decision, not the maximum available | LLM Gateway prompt compression (§8.3) + data product API design |

### 3.3 Data store types

| Store type | Purpose | Reference tech |
|---|---|---|
| **Relational (OLTP)** | Transactional state for pillars and platforms: tickets, incidents, ADRs, audit records | RDS PostgreSQL |
| **Time-series** | Metrics, telemetry, episodic agent memory | TimescaleDB hypertables on RDS, CloudWatch Metrics |
| **Vector + lexical (hybrid retrieval)** | Knowledge layer retrieval — see §4 | pgvector (HNSW) on RDS + OpenSearch (BM25) (D13) |
| **Graph (deferred to Phase 3)** | CMDB relationships, incident causality, code-architecture relationships | Per D14, not deployed by default; Neo4j on EKS or Neptune if needed |
| **Document / blob** | Artifacts, evidence, large payloads | S3 (hot) + S3 Glacier (cold) |
| **Key-value cache** | Working memory, hot reference data, semantic cache for LLM responses | ElastiCache Redis |
| **Stream (event log)** | Inter-component eventing, change-data-capture | MSK Kafka |
| **Metadata catalogues** | Data Contracts Registry, MCP Catalog, Prompt Registry, Asset Inventory | DynamoDB |

### 3.4 Data Contracts Registry

A single programme-level registry that holds one contract per data product. Each contract specifies:

- **Schema** (versioned)
- **Owner** (named team)
- **Freshness SLA** (target staleness and breach action)
- **Access policy** (who, under what conditions, with what governance gate)
- **Lineage** (upstream sources, downstream consumers including agents)
- **Classification** (data sensitivity, PII flags, residency)
- **Decommissioning plan**

Reference tech: DynamoDB + small management UI on Lambda. Schema-as-code in Git; registry is the runtime cache and discovery surface.

### 3.5 Anchor data products

Each service line publishes a small set of anchor data products. All other data products are derived from anchors and inherit their contracts.

| Service line | Anchor entities | Contract owner |
|---|---|---|
| **AMS** | `ticket`, `incident`, `CI` (from CMDB), `runbook`, `resolution_signature` | AMS data team + CMDB owner for CIs |
| **AD** | `spec`, `story`, `repo`, `ADR`, `test_run`, `pipeline_run` | AD platform team |
| **IMS** | `CI` (shared), `telemetry_stream`, `alert`, `patch`, `cost_line` (FOCUS-normalised) | IMS platform team + FinOps team for cost |

### 3.6 Data flows (canonical)

| Flow | Producer | Consumer | Mechanism |
|---|---|---|---|
| Ticket creation | ITSM system (Jira/SNOW) | AMS ingestion (AMS-IP L1) | Webhook + Kafka topic |
| Monitoring alert | Monitoring tools | AMS/IMS ingestion | Webhook + Kafka topic |
| Code commit | GitHub Enterprise | AD ingestion (AD-ACAP L1) | Webhook + Kafka topic |
| Pipeline event | CodeCatalyst / GitHub Actions | AD ingestion | Webhook + Kafka topic |
| Telemetry stream | Cloud providers, OS agents, app instrumentation | IMS ingestion (IMS-IP L1) | Kafka topic + Kinesis Firehose for high-volume |
| Cost data | Cloud billing APIs, SaaS vendors | FinOps ingestion (FOCUS pipeline) | Scheduled extraction, FOCUS-normalised |
| Agent outcome | Any agentic platform | KL + AgentOps | Class B MCP write + telemetry stream |
| Governance decision | Governance Sidecar | Audit Log | Direct write to append-only log |

### 3.7 Data retention

| Data class | Hot retention | Cold retention | Notes |
|---|---|---|---|
| Operational telemetry (metrics, logs) | 90 days | S3 Glacier indefinite | Per regulatory class |
| Transactional records (tickets, incidents, ADRs) | Lifetime in RDS | S3 archive after closure + N years | N varies by data class |
| Agent traces and prompts/responses | 30 days | S3 Glacier (sampled) | Span events redacted at Collector for PII |
| Audit records | 12 months hot in RDS | S3 Glacier indefinite | Append-only, tamper-evident |
| Eval scores and red-team results | 12 months | S3 indefinite | Required for ADLC traceability |
| Lineage history | Full history online | n/a | Required for forensics |

---

## 4. Knowledge architecture

### 4.1 Premise

The Knowledge Layer (KL) is the single substrate agents reason against. It serves retrieval for in-context augmentation, long-term semantic memory for agents (§7), and entity-relationship navigation for high-causality use cases (CMDB, incident chains).

### 4.2 KL components

| Component | Role | Reference tech |
|---|---|---|
| **Embedding store (dense)** | Vector search for semantic similarity | pgvector (HNSW) on RDS PostgreSQL |
| **Lexical store (sparse)** | BM25 keyword search for precise term match | OpenSearch |
| **Reranker** | Re-scores top-K from hybrid retrieval with cross-encoder attention | **Cohere Rerank on Bedrock** (D15) |
| **Knowledge graph (deferred to Phase 3)** | Entity-relationship navigation for causality and structural queries | Decision deferred per D14; Neo4j on EKS or Amazon Neptune |
| **Time-series store** | Episodic patterns, recurrence forecasting inputs | TimescaleDB hypertables |
| **Document store (append-only)** | Raw evidence, large artifacts | S3 + RDS JSONB index |
| **RAG evaluation harness** | Continuous retrieval-quality measurement | RAGAS-style framework |

### 4.3 Retrieval pipeline (production pattern)

Naïve vector-only retrieval fails ~40% of the time in production. The default retrieval pattern:

```
Query → Query transformation (HyDE / decomposition where useful)
      → Hybrid search: BM25 (top-K) ∪ dense vector (top-K)
      → Reranker re-scores the union (top-N out)
      → Optional: knowledge-graph expansion if structural context needed
      → Context assembly with minimum-viable-data discipline
      → LLM Gateway invocation
```

The pipeline lives inside the orchestrator (LangGraph node) of each agentic platform; the stores live in the shared KL.

### 4.4 When to add the knowledge graph

Knowledge graph extraction costs 3–5× more LLM calls than baseline RAG. Per D14, the KG layer is **deferred to Phase 3** and added only if hybrid retrieval hits quality ceilings on the candidate use cases below. Default: don't add it. Add it for use cases where:

- Relationships dominate the question (CI → service → workload dependencies)
- "Why did X cascade to Y" causality is central (incident analysis, root-cause)
- Cross-document entity resolution matters (e.g., the same CI named differently in CMDB and monitoring)

Phase 3 candidate scope (proven on demand, not deployed by default): AMS-01 (CMDB + runbook relationships), IMS-01 (CI dependency graph). Other pillars use hybrid retrieval without KG indefinitely.

Implementation choice deferred until Phase 3 commitment: Neo4j on EKS vs Amazon Neptune.

### 4.5 KL ingestion and freshness

- Writes to the KL go through the **KL Ingest MCP** (Class B, §8.2).
- Every write carries provenance: source data product, timestamp, embedding model + version, redaction applied.
- Embedding freshness is monitored by data observability (§9.4); drift triggers re-embedding workflows in the ML pipeline (§5).
- KL data is partitioned by service line for access control, but federated for cross-line semantic queries.

### 4.6 KL governance

- Every read is authorised by the calling agent's identity (P9).
- Sensitive content is redacted at ingestion (PII detection) and again at retrieval (output filter).
- KL is consulted by data contracts (§3.4) — agents query data products, not raw indexes, even when the underlying store is the KL.

---

## 5. ML & data pipelines

### 5.1 Premise

The programme runs four classes of pipeline. They share infrastructure (EKS, MSK, Kafka, S3, SageMaker) but have different cadences, owners, and SLAs.

### 5.2 Pipeline classes

| Class | Purpose | Cadence | Owner | Reference tech |
|---|---|---|---|---|
| **Ingestion pipelines** | Move raw operational data (tickets, telemetry, alerts, code events) into the foundation stores | Real-time (streaming) and batch | Service-line data teams | MSK Kafka, Kafka Connect, Kinesis Firehose, EventBridge |
| **ML training pipelines** | Train predictive models (MTBF, time-series, pattern clustering) | Scheduled retraining (daily to monthly) | Service-line ML teams | SageMaker Training, SageMaker Model Registry, Step Functions for orchestration |
| **Embedding & KL build pipelines** | Generate embeddings for KL; build BM25 indexes; refresh KG nodes/edges | Streaming for new content; scheduled rebuilds for re-embedding on model changes | KL owner | EKS jobs, Bedrock embedding models, OpenSearch indexer, optional KG extractor |
| **Outcome feedback pipelines** | Capture agent outcomes, evaluate, feed back to drift detection and retraining | Streaming + scheduled | AgentOps team | Kafka, AgentOps event store, RAGAS-style evaluator, Bedrock Evaluations |

### 5.3 Predictive Intelligence System pattern (shared)

Predictive workloads (recurrence forecasting in AMS, capacity forecasting in IMS) follow the same four-layer pattern:

| Layer | Purpose |
|---|---|
| L1 Ingestion | Historical closed cases, time-series data, contextual features |
| L2 Models | MTBF deterministic, time-series ML (Prophet/LSTM), NLP clustering, cascade models |
| L3 Knowledge integration | Retrieval against KL for context; outcome publisher to KL |
| L4 Generation | Forecasts, predictions, push-notify to consuming platforms |

Reference tech is the same in both deployments (AMS-PIS, IMS-PIS): EKS + TimescaleDB + SageMaker + Prophet/scikit-learn + RDS for pattern libraries.

### 5.4 The ADLC Runtime Optimization loop

The Continuous Learning capability (per D11) is implemented as a feedback pipeline:

```
Agent outcomes (success / failure / reversion / customer feedback)
  → Aggregation (per agent, per use case, per service line)
  → Drift detection (output distribution, eval-score regression, tool-call patterns)
  → Eval triggers (run LLM-as-judge / RAGAS on recent samples)
  → Decision: prompt update / model re-route / re-train / no-op
  → Promotion through ADLC Test → Deploy → Operate
  → Audit log entry
```

This pipeline is shared across the three service lines (one AgentOps platform; per-line configurations).

### 5.5 FOCUS billing pipeline (FinOps-specific)

Cloud billing and SaaS billing data is normalised through the **FOCUS (FinOps Open Cost and Usage Specification)** schema before reaching the FinOps store. AI token costs from the per-line LLM Gateways feed the same pipeline as a parallel input stream — token cost is a first-class FinOps dimension (§14).

### 5.6 Pipeline governance

- Pipelines emit lineage events to OpenLineage (§9.4); data observability watches freshness, volume, schema, distribution.
- Pipeline failure does not silently degrade agent quality; data observability raises alerts and agents reading affected data products fail safe per D-P4.

---

## 6. Agent orchestration

### 6.1 Orchestration framework

LangGraph is the single orchestration framework, loaded into AgentCore Runtime (D5). Agents are modelled as explicit state machines with named nodes and edges. Three reasons:

- Explicit state makes reasoning chains observable, debuggable, and testable.
- Checkpointing primitives support human-in-the-loop pauses and resumption (§6.5).
- Edge routing (conditional and definite) makes governance gates first-class control flow, not afterthoughts.

### 6.2 Reasoning strategies

| Strategy | When to use | Where it lands |
|---|---|---|
| **ReAct (reason-act loop)** | Default for tool-using agents — reasoning and tool calls interleave naturally | AMS-05 resolution, IMS-05 remediation, AD-03 code generation |
| **Plan-then-Execute** | Multi-step procedures with clear sequence; cost of mid-stream replanning is high | AMS-05 change execution, IMS-03 patch waves, IMS-05 DR orchestration, AD-03 refactoring |
| **Reflexion (self-critique)** | High-stakes actions where a self-review step reduces error materially | AMS-05 production changes, IMS-08 security response, IMS-05 prod-critical remediation |
| **Chain-of-Thought** | Analysis without action; cost-efficient | AMS-04 / IMS-04 predictive analysis, AMS-03 triage explanation |

### 6.3 Multi-agent topology

When a single agent isn't enough, the default topology is **supervisor + specialists**:

- One supervisor agent decomposes the goal and routes subtasks
- N specialist agents handle defined slices (e.g., RCA specialist, runbook specialist, communication specialist)
- Specialists return structured artifacts; supervisor composes the final result

Alternative topologies (sequential, concurrent, handoff, group chat) are used where the problem genuinely fits them. The supervisor pattern is the default because it preserves a single point of governance and audit.

### 6.4 The 11-node resolution graph (canonical agentic platform pattern)

Agentic platforms (AMS-ARP, IMS-AOP, AD-ACAP autonomous delivery, IMS-SPP response, IMS-CPP execution) implement an 11-node state graph:

| Stage | Nodes |
|---|---|
| **Understanding** | Domain identification, intent classification, symptom/action understanding |
| **Planning** | Context assembly (hybrid RAG, §4.3), resolution planning (strategy per §6.2) |
| **Execution** | OPA governance gate, human interaction (HITL gate, §6.5), action execution (tool calls via §8) |
| **Learning** | Verification, downstream system update (ITSM, CMDB, etc.), knowledge feedback to KL |

The same graph applies across service lines; service-line variation is in the tool set, the domain ontology, and the action space.

### 6.5 Human-in-the-Loop (HITL) and checkpointing

Agents at autonomy tiers L1–L4 require explicit human gates. The architecture supports four HITL patterns:

| Pattern | Behaviour | When to use |
|---|---|---|
| **Synchronous block** | Agent pauses, surfaces a proposed action, waits for human approval before continuing | High-stakes, real-time decisions (prod change in business hours) |
| **Async notify-and-wait** | Agent persists state, notifies human via channel, resumes when human responds | Long-running tasks; cross-time-zone reviews |
| **Trust-but-verify** | Agent acts; human reviews after the fact within a defined window | Low-risk, reversible actions where speed matters |
| **Kill-switch** | Programme-wide or per-tool toggle that halts all matching agents | Incident response; rapid policy change |

All four are implemented on the LangGraph checkpointer — state persists durably, the agent can resume from the exact checkpoint, no lost context. The Governance Sidecar (§12) determines which pattern applies per action class.

### 6.6 Cross-RA agent flows: A2A

When agents in different service lines collaborate, the boundary is crossed via the **A2A protocol** (Agent-to-Agent, Linux Foundation, 2025), not MCP. The decision rule:

- **A2A** if the receiver is a durable agent with its own task lifecycle and decision-making authority
- **MCP** if the receiver is a deterministic tool or service (create ticket, write record, query data)

A2A primitives: Agent Cards (capability advertisement at `/.well-known/agent-card.json`), task lifecycle states (submitted → working → input-required → completed / failed / canceled / rejected), HTTP + SSE + JSON-RPC 2.0 transport. Trace context propagates in message metadata.

### 6.7 Determinism and stochasticity controls

Agents are non-deterministic. Forensics, audit, and regulated environments require:

| Control | Mechanism |
|---|---|
| Model version pinning | Bedrock model-version pins in LLM Gateway (§8.3) |
| Temperature / top-p logging | OTel GenAI span attributes (D6) |
| Seed capture | Where the model supports it; otherwise stochasticity is acknowledged in audit |
| Reproducibility metadata | Audit log records model + temperature + seed + prompt SBOM + retrieved context |
| Replay capability | Reconstruct from audit + execution trace; deterministic tools replay exactly, LLM calls replay best-effort |

---

## 7. Agentic memory

### 7.1 Premise

Agents need multiple memory types beyond the LLM context window. Memory-less agents lose ~26% accuracy compared to persistent-memory agents in production benchmarks. The architecture defines five memory tiers and where each lives.

### 7.2 Five memory tiers

Memory framework: AgentCore Memory (D17) — managed by AWS, integrates with AgentCore Runtime, covers short-term and long-term tiers natively. Mem0 / Letta considered only if future benchmarks prove AgentCore materially weaker for our workloads.

| Tier | Purpose | Lifetime | Reference tech |
|---|---|---|---|
| **Working memory** | Current step's variables, intermediate reasoning | Single step | In-context + ElastiCache Redis for cross-step state within a session |
| **Short-term (session)** | Current task / conversation context | Hours to days | AgentCore Memory short-term |
| **Long-term semantic** | Facts, preferences, learned patterns, user/system profiles | Persistent | AgentCore Memory long-term + KL/pgvector for federated reuse |
| **Long-term episodic** | Sequences of events: what happened, in what order, under what conditions | Persistent | TimescaleDB hypertable + AgentCore Memory summarisation strategy |
| **Procedural** | Runbooks, workflows, repeated action patterns | Persistent | RDS + S3 (runbook store) — shared with AMS-01 / IMS-01 |

### 7.3 Context window management

The LLM context window is a scarce resource. The architecture treats context as actively managed:

- **Summarisation passes** compress long conversation history at session boundaries.
- **Selective retrieval** uses the minimum-viable-data principle (D-P6) — top-N reranked chunks, not raw top-K.
- **Eviction** is importance-weighted, not LRU.
- **Pre-flight token budgeting** estimates cost before submission; rejects requests that would exceed budget without explicit approval (§14).

Context management is a LangGraph node in every agent's graph, not an ad-hoc concern.

### 7.4 Cross-service-line memory federation

| Tier | Federation rule |
|---|---|
| Working | Per-agent, no federation |
| Short-term | Per-agent, no federation |
| Long-term semantic | Federated through KL; an AMS pattern can inform an IMS agent |
| Episodic | Scoped to service line by default; cross-line queries via KL |
| Procedural | Shared at foundation pillars (AMS-01 runbooks, IMS-01 baselines) |

### 7.5 Memory governance

- Every memory write passes through the Governance Sidecar (memory poisoning is a known OWASP Agentic top-10 threat, §11.2).
- Memory deletion is auditable (Operate phase of ADLC, §12.2).
- Sensitive content is redacted before persistence; PII detection runs on extraction.
- Memory is observable: writes, reads, evictions, and decay all emit telemetry (§9.3).

---

## 8. Integration plane

### 8.1 Two integration protocols

The programme uses two integration protocols, with clear allocation:

| Protocol | Role | Direction |
|---|---|---|
| **MCP (Model Context Protocol)** | Agent-to-tool | Agents call MCP servers; servers expose typed tools, resources, prompts |
| **A2A (Agent-to-Agent)** | Agent-to-agent across runtimes or organisational boundaries | One agent invokes another's task lifecycle |

Within a service line, in-process orchestration (LangGraph edges and nodes) is the default and neither protocol is involved. MCP and A2A apply at the platform boundary or above.

### 8.2 MCP classes

| Class | Transport | Examples | Used by |
|---|---|---|---|
| **Class A — Local stdio** | stdio | Filesystem MCP, local Git, test runner | AD only (Kiro on developer laptop) |
| **Class B — Remote internal HTTP** | Streamable HTTP | KL Ingest/Query, Governance Sidecar, ITSM, CodeCatalyst, Bedrock | All three service lines |
| **Class C — Third-party SaaS** | Streamable HTTP via Egress Gateway | Jira, Confluence, Slack, PagerDuty | All three service lines |

### 8.3 The integration plane (programme-level)

A centralised access plane prevents duplication of catalogue, approval, egress, identity, and observability across three service lines. The plane is **AgentCore Gateway-backed** with custom logic where needed.

| Component | Purpose | Reference tech |
|---|---|---|
| **MCP Catalog** | Searchable registry of approved Class B and Class C MCP servers; uses official MCP Registry schema (Linux Foundation) | DynamoDB + small UI on Lambda; private instance per P3 |
| **MCP Approval Workflow** | Onboarding pipeline: security review, data residency check, owner assignment, tool-description poisoning review, SBOM capture | Step Functions + reviewer UI |
| **MCP Gateway** | Single egress for Class B internal and Class C external MCP traffic; OAuth 2.1, DLP, audit, rate limiting, kill-switches, virtual MCP servers, W3C Trace Context propagation | AgentCore Gateway (managed) + API Gateway + Lambda for custom logic |
| **MCP Identity Broker** | Propagates user/service identity; mints short-lived credentials; outbound OAuth flows | AgentCore Identity + IAM Identity Center + STS |
| **MCP Health & Observability** | Tracks MCP availability, latency, error rates, per-tool invocation counts | AgentCore Observability + CloudWatch + X-Ray |

#### Hardening primitives

- **OAuth 2.1** at the Gateway, per MCP spec 2025-03-26 — resource server / authorization server separation.
- **Kill-switches** — programme-level toggles that disable a tool class instantly, enforced as OPA policies at the Gateway.
- **Virtual MCP servers** — compose curated tool subsets per team or agent without redeploying servers. Example: AMS resolution agents see a read-mostly virtual MCP; AMS change agents see one with write tools; same physical servers behind both.
- **Tool-description poisoning review** — LLM-readable tool descriptions are an attack surface (OWASP ASI 2026, indirect prompt injection). Approval Workflow runs automated scan + manual review of every server's exposed descriptions before catalogue admission.
- **Trace context propagation** — Gateway propagates `traceparent` headers to all MCP servers (P10). Disconnected MCP traces are a defect.

### 8.4 LLM Gateway (per service line)

The LLM Gateway is the single chokepoint between agents and foundation models for its service line. One per service line (D4) to align with cost allocation and data residency. All three Gateways share the same architecture; configurations differ.

| Responsibility | What it does |
|---|---|
| **Multi-model routing** | Routes to Claude Sonnet for reasoning, Amazon Nova for high-throughput code/text, embedding models for KL writes — by task type, latency budget, cost ceiling |
| **Fallback chains** | Primary → cheaper model → semantic cache → 503; triggered by 429 / 5xx |
| **Token-aware rate limiting** | Per-team, per-project, per-key token budgets (not just request count) |
| **Circuit breakers** | Trip on cost velocity, repeated prompts, error rate, growing context size |
| **Prompt compression** | Strip redundancy while preserving meaning (up to ~5× input-token reduction on long prompts) |
| **Semantic caching** | Recurring queries hit cache, keyed by embedding similarity |
| **Cost attribution** | Token cost tagged with use case, team, agent, request ID; feeds §14 |
| **Determinism controls** | Model version pinning, temperature/top-p logging, seed capture (§6.7) |
| **PII guard** | Pre-flight scan; redact-or-reject policy per data class |
| **Trace context** | OTel GenAI Semantic Conventions on every span (D6) |

Reference tech: built on Bedrock Cross-Region Inference with a custom orchestration layer. Alternatives evaluated and not chosen as primary: LiteLLM (Python latency overhead at scale), Kong AI Gateway (heavyweight for the scope), Portkey (managed; adds a network hop).

### 8.5 Outbound integration patterns

| Pattern | Used for | Mechanism |
|---|---|---|
| **Tool call (MCP)** | "Do this discrete action" | Class B or C MCP through Gateway |
| **Agent invocation (A2A)** | "Take responsibility for this task and report back" | A2A endpoint, durable task lifecycle |
| **Data product read** | "Give me this entity's current state" | KL query / data product API through MCP |
| **Event publication** | "Inform interested consumers something happened" | Kafka topic via service-line producer |
| **Scheduled extraction** | "Pull SaaS data on a cadence" | Step Functions + Lambda + Class C MCP |

---

## 9. Observability

### 9.1 Four observability domains

Observability is four distinct domains with different signals, tools, owners, retention, and SLOs. Conflating them into a single stack underserves at least three of them.

| Domain | Monitors | Standard | Primary tools | Owner |
|---|---|---|---|---|
| **Infrastructure** | EKS, RDS, Kafka, Lambda, network, storage | OTel core | CloudWatch + X-Ray + OTEL collector | Platform team |
| **Application** | API latency, business KPIs (MTTR, deploy frequency), user journeys | OTel core | CloudWatch Application Signals + OTEL | Service-line product owner |
| **AI** | Agent reasoning, LLM calls, tool calls, memory ops, eval scores, drift | **OTel GenAI Semantic Conventions** (D6) | AgentCore Observability + Langfuse + CloudWatch GenAI page | AgentOps team |
| **Data** | KL freshness, embedding drift, data-contract SLA adherence, lineage | OpenLineage | Glue Data Quality + OpenLineage on Marquez (D18) | Data architecture team |

### 9.2 Infrastructure & application observability

Standard practice: RED metrics (Rate, Errors, Duration) at API boundaries, USE metrics (Utilisation, Saturation, Errors) at resource level. CloudWatch is the metrics-and-logs surface; X-Ray for traces; OTEL Collector on EKS aggregates and forwards. Per-service SLOs published in CloudWatch Application Signals.

Retention: metrics 15 months, logs 90 days hot + S3 Glacier long-term, traces 30 days.

### 9.3 AI observability (the agentic-specific domain)

A traditional APM trace ends at the API boundary. An agent trace continues *inside* the agent — into the LLM call (model, prompt, tokens), into the tool call (which MCP server, arguments, result), into the memory write (what was extracted, what was retained). Without this domain, agent failures are unexplainable.

#### Signals captured

OpenTelemetry GenAI Semantic Conventions (D6) define the canonical span attributes:

- `gen_ai.operation.name` — chat, embeddings, create_agent, invoke_agent
- `gen_ai.provider.name`, `gen_ai.request.model`, `gen_ai.response.model`
- `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`
- Agent Framework Conventions layer Tasks, Actions, Agents, Teams, Artifacts, Memory as first-class span types

#### Prompt and response capture

Prompts and responses are stored as **span events**, not span attributes. Reasoning: attributes are always indexed and exposed in backends; events can be filtered or dropped at the Collector for PII compliance.

#### Tools

| Tool | Role |
|---|---|
| **AgentCore Observability** | Managed primary; emits OTel GenAI spans for runtime, memory, gateway, identity, built-in tools |
| **Langfuse** | Self-hosted on EKS; deeper prompt/response capture, eval annotation, side-by-side prompt comparison |
| **CloudWatch GenAI Observability** | Console surface for AgentCore-emitted data |
| **Galileo Luna or equivalent** | Cost-efficient deterministic evaluators for high-traffic 100%-eval scenarios |

Retention: spans 30 days hot; prompt/response content sampled and retained per data class; eval scores 12 months.

### 9.4 Data observability (the data-specific domain)

Five canonical pillars (Monte Carlo's definition):

| Pillar | Question answered |
|---|---|
| **Freshness** | Is this data product current within its SLA? |
| **Volume** | Did the expected volume of records arrive? |
| **Schema** | Has the schema changed in a way that breaks consumers? |
| **Distribution** | Are values within expected ranges and distributions? |
| **Lineage** | What upstream sources fed this; what downstream consumers depend on it? |

Plus AI-specific signals: **embedding distribution drift**, **retrieval-quality regression** (RAGAS-style), **KL index staleness**.

Tool stack (D18): AWS-native — Glue Data Quality for column-level rules, OpenLineage on Marquez for lineage, custom rules on EKS for AI-specific signals. Open standards (OpenLineage); no commercial license commitment; team owns the stack.

### 9.5 End-to-end trace propagation (P10)

The four domains are correlated by W3C Trace Context propagated through every hop:

```
User/dev → Orchestrator (LangGraph) → LLM Gateway → LLM provider → MCP Gateway → MCP server → external system → return path
```

Enforced as platform rules:

- MCP Gateway propagates `traceparent` headers to all MCP servers.
- A2A protocol calls carry trace IDs in message metadata.
- AgentCore Observability custom headers extend tracing into managed components.
- LLM Gateways inject trace context into Bedrock invocations.

Without this, MCP and A2A boundaries fragment traces. With it, a multi-agent multi-tool failure traverses the full call chain in a single view.

### 9.6 SLOs

Each platform publishes SLOs against the domains relevant to it:

| Platform class | Infrastructure SLO | Application SLO | AI SLO | Data SLO |
|---|---|---|---|---|
| Reactive intelligence (AMS-IP, IMS-IP) | Service uptime, latency | Per-pillar processing time | Agent success rate, latency | KL freshness for read |
| Predictive (AMS-PIS, IMS-PIS) | Pipeline reliability | Forecast availability | Model accuracy, drift threshold | Training data freshness |
| Agentic (ARP, AOP, CPP, SPP, ACAP) | Runtime availability | Resolution rate | Tool-call correctness, eval score, cost per invocation | Retrieved-context freshness |
| Governance / monitoring (CMP, FAG) | Service uptime | Audit coverage | Eval-pipeline cadence | Data-contract SLA adherence |

---

## 10. Reliability & resilience

### 10.1 Premise

Reliability for agentic systems combines classical patterns (multi-AZ, health checks, retries) with agent-specific patterns (cost-velocity circuit breakers, kill-switches, fail-safe-not-bypass behaviour). The classical patterns are necessary but not sufficient.

### 10.2 Classical reliability patterns

| Pattern | Where applied |
|---|---|
| **Multi-AZ** | RDS Multi-AZ for transactional stores; EKS node groups across AZs; MSK Kafka multi-AZ brokers |
| **Auto-scaling** | HPA on EKS; AgentCore Runtime auto-scales; Lambda concurrent execution limits |
| **Health checks** | Application-level health endpoints; ELB / target-group health; CloudWatch alarms |
| **Backup & recovery** | RDS automated backups; S3 versioning; daily snapshot of stateful stores; documented RTO/RPO per platform |
| **Circuit breakers** | At LLM Gateway and MCP Gateway; trip on error rate, latency, cost velocity |
| **Retries with exponential backoff** | All MCP and LLM calls; idempotent operations only |

Multi-region is Phase 4+ scope (§15.5); Multi-AZ from Phase 1.

### 10.3 Agent-specific reliability patterns

| Pattern | What it prevents | Where it lives |
|---|---|---|
| **Cost-velocity circuit breakers** | Runaway agent loops that exhaust monthly budget in hours | LLM Gateway (§8.4) |
| **Repeated-prompt detection** | Same prompt cycling means the agent is stuck | LLM Gateway |
| **Growing-context detection** | Context size ballooning means context isn't being managed | LLM Gateway + agent's context node (§7.3) |
| **Tool-call rate limits per agent** | Tool flooding from a misbehaving agent | MCP Gateway |
| **Programme-wide kill-switches** | Bad change to a tool or prompt needs immediate halt | OPA policies at MCP Gateway |
| **HITL fallback** | Agent can't decide — escalate to human, don't guess | LangGraph checkpointer + Governance Sidecar |
| **Fail-safe-not-bypass** | When dependencies are down, governance must remain enforced (P8) | OPA gate; default-deny when sidecar unavailable |

### 10.4 Degradation behaviour

| Component down | Effect | Behaviour |
|---|---|---|
| **Governance Sidecar** | No agent action can be authorised | All write-path agent actions stop; read-only agents continue; clear alert |
| **MCP Gateway** | No tool calls possible | Agents pause at execution node; checkpointer holds state; resume on recovery |
| **LLM Gateway** | No model calls possible | Agents pause; degraded mode for cached responses on idempotent queries |
| **AgentCore Memory** | Long-term memory unavailable | Agents continue with working+short-term only; quality degrades; logged |
| **KL retrieval** | Hybrid retrieval down | Agents continue with degraded context (no RAG); reasoning quality drops; logged |
| **AgentOps** | No telemetry capture | Agents continue; telemetry buffered in OTel Collector; backfilled on recovery |
| **Audit Log** | No write-path possible | All write actions block; full read-only operation |

The pattern: **graceful degradation for read-side, hard stop for write-side**. Write actions without a working audit log or governance gate are not permitted, ever.

### 10.5 Disaster recovery

| Asset class | RPO target | RTO target |
|---|---|---|
| Audit log | Zero (synchronous replication) | Minutes (read-replica failover) |
| Transactional state (tickets, incidents, etc.) | Seconds (Multi-AZ) | Minutes (Multi-AZ failover) |
| KL embeddings / indexes | Hours (rebuildable from sources) | Hours (rebuild pipeline) |
| Agent traces / observability | 24 hours (sampled retention) | Best-effort |
| Cached model responses | Lossy (semantic cache) | None |

---

## 11. Security & threat response

### 11.1 Threat model anchors

Agentic systems face a different threat surface than traditional applications. The architecture anchors to four external frameworks:

| Framework | Role |
|---|---|
| **OWASP Top 10 for Agentic Applications (December 2025)** | Canonical threat catalogue for autonomous agents |
| **AT-FAA / SHIELD framework (2025)** | Threat taxonomy with mitigation strategies (memory poisoning, intent hijacking, autonomous privilege escalation) |
| **MITRE ATLAS** | Adversarial technique catalogue including AI-specific additions |
| **NIST AI RMF** | Governance methodology |

### 11.2 OWASP Agentic top threats (summary)

Each requires architectural mitigation, not just monitoring:

| Threat | Mitigation home |
|---|---|
| **Indirect prompt injection** | Tool-description poisoning review (§8.3); content scanners on external inputs reaching agents; prompt-injection detector in IMS-SPP |
| **Memory poisoning** | Memory writes governed by Sidecar (§7.5); memory-anomaly detector |
| **Intent hijacking** | OPA gates per action class; HITL on high-stakes actions (§6.5) |
| **Tool misuse / unauthorised tool calls** | Virtual MCP servers (§8.3) scoping which agents see which tools; identity-aware authorisation per call |
| **Autonomous privilege escalation** | Least-privilege agent identities; scoped IAM roles; kill-switches |
| **Excessive autonomy without oversight** | ADLC authority boundaries (§12.2); maturity-staged autonomy progression (L1 → L5) |
| **Resource exhaustion (cost-based DoS)** | Cost-velocity circuit breakers (§10.3) |
| **Data exfiltration via tool calls** | DLP at MCP Egress Gateway; per-call audit; outbound content inspection |

### 11.3 Security controls deployment

| Control class | Mechanism |
|---|---|
| **Identity & access** | AWS IAM Identity Center; scoped IAM roles per agent; no static credentials in agents; AgentCore Identity for outbound OAuth |
| **Network** | VPC isolation; private subnets for agents; PrivateLink to AWS services; egress-only via the MCP Egress Gateway |
| **Data protection** | KMS encryption at rest; TLS in transit; per-tenant key separation where required; redaction at ingestion and retrieval |
| **Secrets management** | Secrets Manager; short-lived credentials minted by AgentCore Identity; no secrets in prompts |
| **Vulnerability management** | Inspector for container images; dependency scanning in CI/CD; signed container images |
| **Detection** | Security Hub; GuardDuty; agent-runtime telemetry for prompt-injection / memory-poisoning / tool-misuse signals |

### 11.4 Continuous red-teaming

Integrated into the ADLC Test & Release phase (§12.2):

- Synthetic adversarial inputs generated against staging copies of every agentic platform
- Multi-modal and MCP-based attack scenarios (per OWASP ASI 2026)
- RL-trained adversarial agents for high-stakes platforms
- Drift in attack-success-rate triggers re-evaluation before promotion
- Tools: Galileo Luna for high-coverage behavioural scoring; LLM-judge ensembles for nuanced evaluation

### 11.5 Security posture progression

| Phase | Posture |
|---|---|
| **Phase 1 (Trust foundation)** | Baseline controls deployed; identity, network, KMS, secrets, audit; security observability live |
| **Phase 2 (Orchestration)** | Detection extended to agent runtimes; OWASP ASI 2026 control set; tool-poisoning review live |
| **Phase 3 (Scale)** | Continuous red-teaming integrated into CI/CD; vulnerability auto-remediation for routine cases |
| **Phase 4 (Autonomy)** | Autonomous security posture (IMS-SPP higher tiers); self-healing security fabric; zero-trust validation continuous |

---

## 12. Governance

### 12.1 Two governance layers

Governance for agentic systems sits at two layers, both required:

| Layer | Role | Location |
|---|---|---|
| **Runtime governance** | Per-action authorisation: "is this agent allowed to take this action right now, on this resource, on behalf of this identity?" | Governance Sidecar (OPA), gates inside the agent execution graph |
| **Lifecycle governance (ADLC)** | Per-agent oversight: "has this agent been planned, evaluated, approved, deployed, monitored, and decommissioned correctly?" | Programme-level lifecycle process |

### 12.2 Agent Development Lifecycle (ADLC)

Every agent built in any service line follows six phases with two inner loops.

#### Six phases

| Phase | Output | Owner |
|---|---|---|
| **Plan** | Agent goals, authority boundaries, risk posture, data classification, threat model | Service-line architect + Governance authority |
| **Code & Build** | Prompts, tool wiring, memory configuration, eval design, behaviour specs | Service-line build team |
| **Test & Release** | LLM-as-judge evals, red-team report, drift baselines, pre-release checks | AgentOps team (§13) + Governance |
| **Deploy** | Champion-challenger, canaries, kill-switch wiring, rollback plan | Platform team |
| **Operate** | Runtime governance, identity propagation, audit trail, HITL gates | Platform team + service-line operators |
| **Monitor** | Hallucination metrics, drift detection, fairness audits, decommissioning | AgentOps + service-line operators |

#### Two inner loops

| Loop | Purpose | Cadence |
|---|---|---|
| **Experimentation** | Build-time iteration on prompts, tools, memory, reasoning strategy; evaluated against frozen test sets | Per-build |
| **Runtime Optimisation** | Production outcomes → drift signals → eval triggers → retraining / re-prompt decisions → re-deploy (D11) | Continuous (online) + scheduled (offline) |

#### Mandatory artifacts per phase

| Artifact | When produced | Where stored |
|---|---|---|
| Agent registration record | Plan | MCP Catalog + Audit Log |
| Behaviour spec | Plan | KL + Audit Log |
| Prompt SBOM (what's in the prompt, which tools it grants) | Code & Build | Prompt Registry (§13.4) |
| Eval results + LLM-as-judge scores | Test & Release | AgentOps store + Audit Log |
| Red-team report | Test & Release | Audit Log |
| Pre-release approval record | Test & Release | Audit Log |
| Decision log (per autonomous action) | Operate | Audit Log |
| Drift detection events | Monitor | AgentOps store + Audit Log |
| Decommissioning record | Monitor | Audit Log |

### 12.3 Governance Sidecar (runtime)

A single programme-level service that every agent calls before executing a write-path action. Implementation: OPA + Rego policies on EKS, append-only audit log to RDS + S3 Glacier.

Inputs to a gate evaluation:

- Agent identity and authority profile
- Proposed action (target system, operation, payload)
- Caller identity (propagated, P9)
- Context (use case, environment, time-of-day, change-window state)

Outputs:

- Allow / Deny / Require human approval (HITL pattern per §6.5)
- Audit record with full inputs and decision rationale
- Reproducibility metadata for forensics (§6.7)

### 12.4 AI code governance (AD-specific, builds on ADLC)

AI-generated code introduces concerns the runtime sidecar doesn't address:

- IP attribution: which developer authored the AI prompt that produced this code?
- License compliance: are dependencies introduced by the AI compatible with target licenses?
- Explainability: can we explain why this code was generated?
- Constitutional constraints: CWE security mappings, license rules, attribution requirements

These are enforced in AD's deployment of the ADLC, with **constitutional SDD** discipline layered on Kiro steering files. No separate tool — steering files + decision logs + the ADLC artifacts above are sufficient.

### 12.5 Compliance evidence

The Audit Log is the single source of compliance evidence. It captures:

- Every governance decision (allow / deny / approval)
- Every autonomous action with its inputs, model + temperature + seed, retrieved context, output
- Every ADLC artifact promotion
- Every memory write affecting future agent behaviour
- Every developer-identity-attributed AI-generated artifact (AD)

**Developer identity handling (D19):** Developer identity is recorded as a **pseudonymised stable ID**, not a raw username or email. Re-identification (mapping the pseudonym back to a real developer) requires **two-person authorisation** through a documented workflow. This satisfies most works-council frameworks (notably EU) while preserving forensic capability for IP, license, and governance investigations. Per-jurisdiction tuning happens during rollout — some jurisdictions may require team-level only.

Retention: 12 months hot in RDS JSONB, indefinite in S3 Glacier. Append-only, tamper-evident.

---

## 13. Quality & evaluation (AgentOps)

### 13.1 Premise

AgentOps is the operational discipline that enforces the ADLC at runtime. It is distinct from MLOps (model training and serving) and LLMOps (single-LLM-call operations). AgentOps adds: multi-step trace correlation, prompt management as code, tool-call correctness scoring, agent trajectory evaluation.

### 13.2 AgentOps scope

| Concern | Capability | Reference tech |
|---|---|---|
| **Distributed tracing** | Multi-step agent traces with reasoning chain visualisation; node-level visibility | AgentCore Observability + Langfuse; OTel GenAI Conventions (D6) |
| **Eval pipeline** | Heuristic + statistical + LLM-as-judge metrics; tool-call correctness; trajectory evaluation | RAGAS for retrieval; LLM-as-judge for output; agent-trajectory eval for multi-step |
| **Prompt management** | Versioned prompts as code; A/B testing; staged rollouts; prompt SBOM | Langfuse prompt registry (self-hosted) + Git for source-of-truth (D16); CI/CD for prompts |
| **Drift detection** | Output distribution drift; eval-score regression; tool-call pattern drift | Galileo Luna for 100%-traffic eval at low cost; AgentCore Memory metrics |
| **Replay & forensics** | Reproduce any agent run from audit log; pin model + temperature + seed | Audit Log (§12.5) + execution trace store |
| **Cost telemetry** | Per-call token cost; per-use-case attribution; per-team budgets | LLM Gateway (§8.4) + cost dashboard feeding §14 |
| **Continuous Learning loop** | Production outcomes → drift → eval → update decisions (D11) | Per service line; shared AgentOps platform |

### 13.3 Evaluation hierarchy

| Eval class | Purpose | Cost | Coverage |
|---|---|---|---|
| **Heuristic** | Fast, cheap baselines (format compliance, length bounds, refusal patterns) | Near-zero | 100% of traffic |
| **Deterministic small-model evaluators (Luna-style)** | Behavioural scoring at 100% coverage | Low | 100% of traffic |
| **LLM-as-a-judge** | Nuanced quality assessment | Moderate | Sampled (5-10% of traffic) |
| **Agent-as-a-judge** | Multi-perspective evaluation for high-stakes outputs | Higher | Sampled (1-5%) |
| **Human annotation** | Last-mile quality checks; gold-standard labelling | High | Targeted (<1%) |

Production eval combines all five classes; the mix is tuned per agent based on cost-quality trade-off.

### 13.4 Prompt management as code

Prompts are critical system components. They get the same rigour as code:

- **Prompt Registry** holds versioned prompts in Langfuse with Git as source-of-truth (D16); versions are signed artifacts
- **Prompt CI/CD** runs the prompt against a golden eval set before allowing promotion
- **A/B framework** routes a slice of traffic to challenger prompts; eval-driven promotion decision
- **Prompt SBOM** records what the prompt contains, which tools it grants the agent access to, which data products it reads
- **Steering files** (Kiro) are the developer-laptop counterpart, distributed centrally

### 13.5 The Continuous Learning loop

Per D11, each service line's Continuous Learning capability is the ADLC Runtime Optimisation loop scoped to that line. Implementation is shared (one AgentOps platform, per-line configurations):

| Service line | Inputs | Outputs |
|---|---|---|
| **AMS Continuous Learning** | Agent outcomes (resolved / escalated / reverted), customer feedback, SLA outcomes | Drift signals → updates for AMS-IP, AMS-ARP, AMS-PIS prompts and models |
| **AD Continuous Learning** | Production defect patterns, perf regressions, PR merge / revert rates, customer-reported issues | Updates for AD-DWP and AD-ACAP agents |
| **IMS Continuous Learning** | Auto-remediation success / failure, MTTR, incident recurrence, cost anomalies, security-incident outcomes | Updates for IMS-IP, IMS-AOP, IMS-CPP, IMS-SPP agents |

### 13.6 Quality SLOs

Each agent declares quality SLOs:

- **Task success rate** — does the agent complete the task as defined in its behaviour spec?
- **Tool selection quality** — does it call the right tool for the situation?
- **Action advancement** — does each step move toward completion?
- **Action completion** — does the trajectory terminate correctly?
- **Cost per invocation** — within budget for the use case?
- **Latency per invocation** — within latency budget?
- **Drift threshold** — output distribution within tolerance vs baseline?

---

## 14. FinOps & cost

### 14.1 Premise

Cloud cost and AI token cost are the same FinOps discipline applied to different units. Cloud cost is dollars per resource-hour; AI cost is dollars per token. Both fluctuate by request, both can spike 100× on bad inputs, both need attribution to use cases and teams.

### 14.2 FinOps framework anchor

The programme adopts the FinOps Foundation 2025 Framework:

- **Three phases** — Inform (visibility, allocation), Optimise (rates, usage), Operate (continuous improvement)
- **Scopes** — Cloud, SaaS, AI, Licensing, Data Centers (programme uses Cloud + SaaS + AI initially)
- **FOCUS** (FinOps Open Cost and Usage Specification) — normalised schema for billing data across providers

### 14.3 Cost dimensions

| Dimension | Source | Granularity |
|---|---|---|
| **Cloud compute** | AWS Cost Explorer (FOCUS-normalised) | Per service, per tag, per team |
| **Cloud data services** | AWS billing | Per database / cluster / topic |
| **Storage** | AWS billing | Per bucket / volume |
| **Network egress** | AWS billing | Per service |
| **AI token cost** | LLM Gateways (per service line) | Per call, per use case, per team, per agent, per request ID |
| **MCP egress (Class C SaaS APIs)** | MCP Egress Gateway | Per tool, per call |
| **SaaS subscriptions** | Vendor billing (manual / API) | Per tool, per team |

All dimensions normalise to FOCUS schema in the FinOps store.

### 14.4 AI cost controls

AI cost is variable per request and can run away faster than cloud cost. Controls live at the LLM Gateway (§8.4):

| Control | Behaviour |
|---|---|
| **Per-key token budgets** | Each application or service identity has a token ceiling; refused beyond it |
| **Per-team budgets** | Aggregate budget pools per business unit; alert at 80%, hard stop at 100% |
| **Per-use-case budgets** | Specific use cases (e.g., a high-volume copilot) get their own caps |
| **Cost-velocity circuit breakers** | Trip on rate of spend, not absolute spend |
| **Cost-aware model routing** | Route to cheaper models when quality permits (Nova for high-throughput, Sonnet for high-reasoning) |
| **Prompt compression** | Reduce input tokens (up to ~5× on long prompts) |
| **Semantic cache** | Recurring queries hit cache; cache-key by embedding similarity |

### 14.5 FinOps lifecycle (per FinOps Framework 2025)

| Phase | Activities | Outputs |
|---|---|---|
| **Inform** | Tag inventory; ingest billing via FOCUS; build cost-allocation views; surface anomalies | Dashboards per scope; chargeback / showback reports |
| **Optimise** | Right-sizing; reserved capacity; commitment management; cost-aware routing decisions; licensing audits | Optimisation recommendations; commitment plans |
| **Operate** | Budget enforcement via Governance Sidecar; continuous anomaly detection; forecast accuracy review | Budget actions; forecast updates; policy adjustments |

### 14.6 Cost attribution flow

```
Agent invocation
  → LLM Gateway records tokens-by-model + use-case + team + agent + request ID
  → Tool calls record per-tool cost (for Class C SaaS-billed tools)
  → Underlying compute / storage / network billed by AWS
  → FOCUS pipeline normalises everything
  → FinOps store joins by request ID for full per-invocation cost
  → Dashboards per scope (cloud, AI, SaaS) and per dimension (team, use case, agent)
```

This is the data flow that turns "we spent $X on cloud" into "the AMS resolution copilot cost $Y per ticket and processed Z tickets last month".

---

## 15. Service-line application

This section describes how the architectural concerns above are applied to each of the three service lines. Per P1, the architecture is shared; service lines differ in their pillar mix, their use-case load, and their platform deployment.

### 15.0 Three-platform pattern (AMS & IMS)

Both AIOps service lines (AMS and IMS) decompose their core agentic surface into three co-evolving platforms backed by the shared Knowledge Layer. AD does not follow this pattern (see §15.3 for AD's hybrid DWP+ACAP+IP model).

| Platform | Role | Operates on | Output | AMS instance | IMS instance |
|---|---|---|---|---|---|
| **Intelligence Platform (IP)** | Reactive enrichment & correlation | Live event streams | Enriched, de-duplicated incidents/alerts in ITSM | AMS-IP | IMS-IP |
| **Predictive Intelligence System (PIS)** | Forward-looking prediction | Time-series telemetry + historical patterns | Failure-window predictions, pre-check tasks | AMS-PIS | IMS-PIS |
| **Agentic Resolution Platform (ARP-class)** | Multi-step resolution reasoning | Tickets created in ITSM, or signals from IP | Governed actions, resolved incidents | AMS-ARP | IMS-AOP (+ IMS-CPP, IMS-SPP for specialised flows) |
| **Knowledge Layer (KL)** | Shared accumulated memory | All three platforms read/write | Embeddings, patterns, outcomes | Shared programme-level | Shared programme-level |

**Why three platforms instead of one:**

- Reactive and predictive workloads have materially different latency requirements (sub-second vs. hours).
- Predictive and agentic platforms evolve at different cadences (PIS retrains scheduled; ARP-class prompts update continuously).
- Independent failure domains matter — a PIS outage must not block IP enrichment, and an ARP outage must not block PIS predictions.
- Different governance models apply to each platform (read-only enrichment ≠ governed write-path actions).
- Phased delivery is preserved — predictive and agentic capabilities can come online after the reactive platform is stable.

**Co-evolution dynamic.** What IP produces at each release directly determines the ARP-class platform's reasoning burden. This is a design fact, not a side effect:

- **IP early-release:** ARP works from symptom-only tickets — high reasoning load, more LLM calls, longer trajectories.
- **IP mid-release (with triage, dedup, enrichment):** advisory fields narrow ARP planning — moderate reasoning load.
- **IP later-release combined with PIS:** pre-check tasks arrive *before* failure — lowest reasoning load, highest precision.

Plan ARP scope and cost envelope around what IP+PIS deliver in the same release window, not in isolation.

**When this decomposition is not the right choice** — small environments where IP and the agentic platform could share a runtime; clients without sufficient data for PIS (predictive capabilities deferred or scoped out); engagements where only signal intelligence and triage are in scope. In those cases collapse to a simpler two-platform or single-platform deployment, but retain the concern separation in §§2–14.

#### 15.0.1 Eight-layer IP reference decomposition

The Intelligence Platforms in both AMS and IMS (AMS-IP and IMS-IP) follow the same eight-layer internal decomposition. Not every engagement needs all eight; smaller scopes may collapse layers (e.g., L4 and L6 if no agents, L5 if no PIS in scope).

| Layer | Name | Function | Typical release phase |
|---|---|---|---|
| **L1** | Data Ingestion | API Gateway, message bus, event normalizer, ITSM webhook classifier (loop prevention) | Phase 1 |
| **L2** | Data Foundation | Relational store, vector store, time-series DB, cache, knowledge-ingest gateway | Phase 1 |
| **L3** | Operational Intelligence | Deduplication, cascade correlation, CMDB enrichment, priority scoring, qualification gate | Phase 1–2 |
| **L4** | AI Services Consumer | LLM Gateway client, RAG pipeline, Resolver Copilot endpoint | Phase 2 |
| **L5** | Predictive Intelligence Consumer | Push-notify + acknowledge protocol with PIS, pre-check scheduler | Phase 2–3 |
| **L6** | Agent Orchestration | LangGraph advisory or executing agents on AgentCore Runtime | Phase 2–3 |
| **L7** | Governance & Audit | OPA gate, audit-log writer, kill-switch enforcement, action approvals | Phase 1 (baseline), Phase 2–3 (enrichment) |
| **L8** | Observability & Telemetry | OTel GenAI emission, AgentOps telemetry, data-observability hooks | Phase 1 (baseline), evolves with platform |

Cross-references: L1–L2 inherit §3 Data architecture; L3 implements pillars from §15.1 (Signal Intelligence, Triage); L4 hits §8.4 LLM Gateway; L5 talks to the PIS via §8 protocols; L6 inherits §6 orchestration; L7 inherits §12 governance; L8 inherits §9 observability.

The L1..L8 model is the canonical "where does this component live inside the IP" answer used during component decomposition (§17).

### 15.1 Capability themes mapped to service-line pillars

Each service line decomposes its work into numbered capability pillars sequenced by maturity (D10). Pillars map to concerns:

| Capability theme | AMS | AD | IMS | Concerns drawn on |
|---|---|---|---|---|
| **Foundation / Discovery** | AMS-01 Knowledge & CMDB Foundation | AD-01 Requirements & Intent, AD-02 Architecture, Design Patterns & Tech Debt | IMS-01 Infra Discovery & Telemetry | §3 Data, §4 Knowledge |
| **Signal Intelligence** | AMS-02 Ticket Signal Intelligence | (n/a) | IMS-02 Infra Signal & Alert Intelligence | §3 Data, §4 Knowledge, §9 Observability |
| **Triage / Routing** | AMS-03 Triage, Routing, Workload & Process Mining | AD-05 Pipeline & Release Orchestration | (in IMS-02) | §6 Agent orchestration |
| **Predictive Intelligence** | AMS-04 Predictive & Pattern | (in AD-01 / AD-02) | IMS-04 Predictive Infrastructure | §5 ML & data pipelines |
| **Act / Agentic** | AMS-05 Agentic Resolution & Change | AD-03 Code Generation, AD-04 Quality & Test | IMS-03 Control Plane, IMS-05a Signal-Driven Auto-Remediation, IMS-05b Workflow-Driven Infra Ops, IMS-06 Endpoint & Specialised, IMS-08 Security Response | §6 Agent orchestration, §7 Memory, §8 Integration |
| **Govern / Compliance** | AMS-06 App Health & Application Compliance | AD-06 AI Governance, IP & Explainability | IMS-07 FinOps, Asset & Licensing; IMS-08 Security Posture | §11 Security, §12 Governance, §14 FinOps |
| **Continuous Learning (ADLC Runtime Optimisation, D11)** | AMS-07 | AD-07 | IMS-09 | §13 Quality, §5 Pipelines |

### 15.2 Pillar → platform → concern mapping (AMS)

AMS deploys **4 platforms** with shared services.

| Pillar | Platform | Concerns inherited (primary) |
|---|---|---|
| AMS-01 Knowledge & CMDB Foundation | Shared KL | §3, §4 |
| AMS-02 Ticket Signal Intelligence | AMS-IP (Intelligence Platform) | §2, §3, §6, §9 |
| AMS-03 Triage, Routing & Workload | AMS-IP | §2, §6, §8, §12 |
| AMS-04 Predictive & Pattern | AMS-IP (reactive) + AMS-PIS (predictive) | §5 |
| AMS-05 Agentic Resolution & Change | AMS-ARP (Agentic Resolution Platform) | §2, §6, §7, §8, §10, §12 |
| AMS-06 App Health & Application Compliance | AMS-CMP (Compliance & Monitoring Platform) | §9, §13 |
| AMS-07 Continuous Learning (ADLC Runtime Optimisation) | Shared AgentOps platform | §5, §13 |

**Pillar boundary notes:**
- *Change & Problem Process Mining* sits in AMS-03 (operational analytics feeding triage, routing, workflow redesign) rather than AMS-01.
- AMS-06 owns **application-layer compliance** (SLA, app health, app-level audit evidence). Infrastructure-layer compliance (zero-trust, security posture, infra audit) belongs to IMS-08. FinOps compliance belongs to IMS-07.
- AMS-04 is intentionally split across two platforms: reactive pattern surfacing (Recurring Pattern Identification, RCA Gap Detection) on AMS-IP; forward-looking forecasting (Predictive Recurrence Forecasting) on AMS-PIS. The split is by latency, not by capability; documented as a deliberate pattern.

| AMS platform | Latency profile | Primary concerns |
|---|---|---|
| **AMS-IP** | Sub-second to seconds | §2 AgentCore Runtime + LangGraph + EKS; §3 RDS + Kafka; §4 hybrid KL retrieval; §6 advisory agents; §9 OTel GenAI; §8 LLM Gateway + MCP |
| **AMS-PIS** | Hours to days | §5 ML pipelines (Prophet, SageMaker); §3 TimescaleDB |
| **AMS-ARP** | Seconds to minutes | §2 AgentCore Runtime + LangGraph; §6 11-node resolution graph, ReAct/Plan-then-Execute/Reflexion strategies, HITL checkpointing; §7 five memory tiers; §8 AgentCore Gateway; §12 OPA gates |
| **AMS-CMP** | Scheduled (minutes to hours) | §9 four-domain observability with emphasis on AI; §13 LLM-as-judge sampling of AMS-ARP, drift detection, periodic red-team |

AMS use-case count: 27 across 7 pillars. Per-pillar: AMS-01 (3), AMS-02 (5), AMS-03 (6), AMS-04 (3), AMS-05 (7), AMS-06 (3), AMS-07 (0 — ADLC Runtime Optimisation).

### 15.3 Pillar → platform → concern mapping (AD)

AD deploys **3 platforms** with shared services. AD-DWP is the only off-AWS surface in the entire programme.

| Pillar | Platform(s) | Concerns inherited (primary) |
|---|---|---|
| AD-01 Requirements & Intent | AD-DWP (interactive) + AD-ACAP (cross-project) | §3, §4 |
| AD-02 Architecture, Design Patterns & Tech Debt | AD-DWP (per-project) + AD-ACAP (portfolio) | §4, §6 |
| AD-03 Code Generation & Delivery | AD-DWP (interactive) + AD-ACAP (autonomous) | §6, §7 |
| AD-04 Quality & Test | AD-DWP (unit) + AD-ACAP (E2E, chaos) | §6, §13 |
| AD-05 Pipeline & Release Orchestration | AD-ACAP | §6, §8 (A2A to IMS-CPP) |
| AD-06 AI Governance, IP & Explainability | AD-IP (Integration Plane / Internal Developer Platform) | §11, §12, §14 |
| AD-07 Continuous Learning (ADLC Runtime Optimisation) | Shared AgentOps platform | §5, §13 |

**Pillar boundary notes:**
- AD-02 owns architecture, design patterns, *and* tech debt analysis. AD-03 stays focused narrowly on producing code. The Design Pattern Recommender and Tech Debt Analyzer agents belong to AD-02.
- AD-01..AD-04 are intentionally split across AD-DWP and AD-ACAP by execution location — interactive work on the developer laptop, long-running and cross-project work on AWS. This split *is* the AD architecture (§5.4 hybrid pattern), not a flaw in it.

| AD platform | Location | Primary concerns |
|---|---|---|
| **AD-DWP** (Developer Workspace, not a deployed platform) | Developer laptop, off-AWS | §2 Kiro IDE; §6 interactive agents; §8 Class A stdio MCPs + Class B/C through AD-IP; §7 short-term memory; §12 steering files + constitutional SDD per D21 |
| **AD-ACAP** (AD Cloud Agent Platform) | AWS | §2 AgentCore Runtime + LangGraph; §3, §4 hybrid retrieval over code/spec; §6 autonomous delivery, refactoring, QA orchestration; §8 cross-RA flows via A2A; §13 outcome publisher |
| **AD-IP** (AD Internal Developer Platform) | AWS | Five-plane IDP model: Developer Control / Integration & Delivery / Resource / Security / Observability. Hosts §12 AD-specific governance (IP scanning, explainability gating, constitutional SDD enforcement per D21), **Powers Approval Workflow** (same Step Functions infra as MCP Approval), **Steering File Distribution** (centrally-managed baseline AGENTS.md and coding standards distributed to all developer laptops), §11 license scanning, §9 audit log writer with pseudonymised developer-identity attribution (D19) |

**Naming note:** AD-DWP drops the "Platform" suffix in prose because it isn't a deployed runtime — it's the Kiro IDE configured on a laptop. The abbreviation AD-DWP is retained for cross-references. ACAP and AD-IP remain platforms.

AD use-case count: 15 across 7 pillars. Per-pillar: AD-01 (3), AD-02 (4 after Design Pattern + Tech Debt move), AD-03 (1, only the Feature Code Generation use case), AD-04 (3), AD-05 (3), AD-06 (1), AD-07 (0 — ADLC Runtime Optimisation).

### 15.4 Pillar → platform → concern mapping (IMS)

IMS deploys **6 platforms** with shared services. The larger count reflects IMS's broader scope (control plane, FinOps, and security each justify dedicated platforms).

| Pillar | Platform | Concerns inherited (primary) |
|---|---|---|
| IMS-01 Discovery & Telemetry Foundation | Shared KL + IMS-CPP foundation | §3, §4 |
| IMS-02 Infra Signal & Alert Intelligence | IMS-IP | §2, §3, §6, §9 |
| IMS-03 Provisioning, Config & Patch Control Plane | IMS-CPP | §6 Plan-then-Execute, §8 A2A endpoint from AD, §12 governance gates |
| IMS-04 Predictive Infrastructure Intelligence | IMS-PIS | §5 ML pipelines |
| IMS-05a Signal-Driven Auto-Remediation | IMS-AOP | §2, §6, §7, §8, §10, §12 |
| IMS-05b Workflow-Driven Infra Ops | IMS-AOP | §6, §8, §12 |
| IMS-06 Endpoint & Specialised Workload Operations | IMS-AOP | §6, §8 |
| IMS-07 FinOps, Asset & Licensing | IMS-FAG | §14 FinOps Framework 2025, FOCUS pipeline, AI token-cost aggregator |
| IMS-08 Security Posture & Threat Response | IMS-SPP | §11 OWASP ASI 2026, AT-FAA/SHIELD, continuous red-team, §12 strictest governance |
| IMS-09 Continuous Learning (ADLC Runtime Optimisation) | Shared AgentOps platform | §5, §13 |

**Pillar structure changes from earlier drafts:**
- *IMS-05* was a single pillar with 16 use cases; it split cleanly into **IMS-05a Signal-Driven Auto-Remediation** (reactive, latency-sensitive — auto-remediation, self-healing services, traffic control, middleware, predictive reliability) and **IMS-05b Workflow-Driven Infra Ops** (humans/schedule/business-event-driven — DR orchestration, backup, batch remediation, Go/No-Go, autonomous inventory, dynamic workload placement, Azure Local). Both deploy on IMS-AOP — the split is at pillar level, not platform level.
- *IMS-06* renamed to **Endpoint & Specialised Workload Operations** to be honest that it is a deliberate "small named pillar" for capabilities that share the agentic ops execution pattern but have domain-specific characteristics (endpoint, radio interference, retail inventory, CV quality/productivity). All four still run on IMS-AOP.
- "Security Architecture, IAM & Threat Modeling" (Foundation-stage) sits in **IMS-08**, not IMS-01. IMS-01 is now strictly infra discovery and telemetry foundation; security strategy belongs with the security team from threat-modelling onward.
- "Hybrid Cloud Tech Debt Analysis" moved from IMS-04 to **IMS-07** — it produces a quantified inventory of debt, an asset/audit activity rather than a forecast.
- Two workbook use cases split across pillars:
  - "Predictive Reliability & FinOps Autonomy" → Predictive Reliability stays in **IMS-05a**; FinOps Autonomy moves to **IMS-07**.
  - "SLA Monitoring, Reporting & Credit Analysis" → real-time SLA Monitoring stays in **IMS-02**; SLA Reporting & Credit Analysis moves to **IMS-07**.

| IMS platform | Latency profile | Primary concerns |
|---|---|---|
| **IMS-IP** | Sub-second to seconds | §2 AgentCore Runtime + LangGraph; §3 RDS + Kafka + TimescaleDB dual hypertable; §4 hybrid KL retrieval; §9 OTel GenAI |
| **IMS-PIS** | Hours to days | §5 ML pipelines (Prophet, LSTM, cascade models); §3 TimescaleDB |
| **IMS-CPP** | Scheduled / change-driven | §2 AgentCore Runtime; §6 Plan-then-Execute for patch waves; §8 A2A endpoint, Class B MCPs; §12 governance gates; §10 separation from IMS-AOP for failure isolation |
| **IMS-AOP** | Seconds to minutes | §2 AgentCore Runtime + LangGraph; §6 11-node resolution graph with Reflexion for high-stakes, supervisor + specialists topology; §7 five memory tiers; §8 AgentCore Gateway; §12 OPA gates. Hosts IMS-05a, IMS-05b, IMS-06. |
| **IMS-FAG** | Scheduled (daily) | §14 FOCUS data ingestion, AI token-cost aggregator across all RAs, tagging engine, invoice reconciliation, licensing automation, Inform → Optimise → Operate dashboards |
| **IMS-SPP** | Variable | §11 detection layer with prompt-injection / memory-poisoning / tool-misuse monitors, vulnerability scanning, governed response, continuous red-team, audit layer; highest governance tier |

IMS use-case count: 50 across **10 pillars**. Per-pillar (post-triage):
- IMS-01 (1), IMS-02 (5), IMS-03 (7), IMS-04 (6 after Tech Debt move out)
- IMS-05a (8 — 7 from original IMS-05 reactive set + 1 from Predictive Reliability split)
- IMS-05b (8 — workflow-driven set, including DR Orchestration, Backup, Batch Remediation, Go/No-Go, Self-Building Agent Crew, Inventory, Dynamic Workload Placement, Azure Local HCI)
- IMS-06 (4 — Endpoint Health, Radio Interference, Retail Inventory, CV Quality)
- IMS-07 (6 — original 3 + Tech Debt + FinOps Autonomy + SLA Reporting & Credit Analysis)
- IMS-08 (7 — original 6 + Security Architecture/IAM/Threat Modeling moved from IMS-01)
- IMS-09 (0 — ADLC Runtime Optimisation)

### 15.5 Programme phased rollout

Phasing applies across all three service lines:

| Phase | Theme | What ships | Capability pillars active |
|---|---|---|---|
| **Phase 1: Trust foundation** | Data, governance, observability, identity | §3 Data architecture, §4 KL substrate, §8 MCP plane, §12 Governance Sidecar + ADLC, §9 four-domain Observability, §13 AgentOps platform, §10 baseline reliability, §11 baseline security | AMS-01, AD-01, AD-02, IMS-01 |
| **Phase 2: Orchestration** | Reactive intelligence, advisory agents | §2 agent runtimes per line, §8 LLM Gateways, reactive pillars on AgentCore + LangGraph | AMS-02, AMS-03, IMS-02, AD-03 (assistive only) |
| **Phase 3: Scale** | Predictive intelligence + governed action execution | §5 predictive pipelines, §6 multi-step agentic platforms with governed actions; KG re-evaluated for AMS-01/IMS-01 (D14) | AMS-04, AMS-05 (L2-L3), IMS-04, IMS-05a/05b (L2-L3), AD-04, AD-05 |
| **Phase 4: Autonomy** | Self-healing + closed-loop learning | §13 Runtime Optimisation loop live across all RAs; higher autonomy tiers (L4-L5); advanced governance; multi-region considered (D22) | AMS-05 L4/L5, AMS-06, AMS-07, IMS-05a/05b L4/L5, IMS-06, IMS-07, IMS-08, IMS-09, AD-06, AD-07 |

Multi-region is Phase 4+ scope; Multi-AZ from Phase 1.

### 15.6 Use-case coverage

Every use case from the catalogue maps to one primary pillar; every pillar maps to a platform.

| Service line | Use cases | Pillars | Platforms |
|---|---|---|---|
| AMS | 27 | 7 | 4 + shared |
| AD | 15 | 7 | 3 + shared |
| IMS | 50 (after splits) | 10 | 6 + shared |
| **Total** | **92** | **24** | **13 + shared** |

Per-pillar load (post-triage):
- **AMS:** AMS-01 (3), AMS-02 (5), AMS-03 (6 — gained Change & Problem Process Mining), AMS-04 (3), AMS-05 (7), AMS-06 (3), AMS-07 (0 — ADLC Runtime Optimisation, consumed by other pillars)
- **AD:** AD-01 (3), AD-02 (4 — gained Design Pattern Recommender + Tech Debt Analyzer), AD-03 (1 — only Feature Code Generation use case remains), AD-04 (3), AD-05 (3), AD-06 (1), AD-07 (0 — ADLC Runtime Optimisation)
- **IMS:** IMS-01 (1 — Security Architecture moved to IMS-08), IMS-02 (5), IMS-03 (7), IMS-04 (6 — Tech Debt moved to IMS-07), IMS-05a (8 — signal-driven set), IMS-05b (8 — workflow-driven set), IMS-06 (4 — endpoint + specialised), IMS-07 (6 — gained Tech Debt + FinOps Autonomy + SLA Reporting), IMS-08 (7 — gained Security Architecture/IAM), IMS-09 (0 — ADLC Runtime Optimisation)

Note: Two workbook use cases were split during triage ("Predictive Reliability & FinOps Autonomy" and "SLA Monitoring, Reporting & Credit Analysis"), each becoming two use case records. Workbook v5 will reflect these splits; total count remains 92 distinct capability use cases when the original bundles are counted as their constituent parts.

No use case is orphaned. The three Continuous Learning pillars have no dedicated use cases by design — they are platform-tax (D11), sized against the AgentOps shared platform.

### 15.7 Cross-service-line flows

Two protocols, with the allocation rule from §8 (A2A for agent-to-agent, MCP for agent-to-tool):

| Touchpoint | Direction | Protocol |
|---|---|---|
| AD pipeline triggers infra provisioning | AD-ACAP → IMS-CPP | A2A |
| AD pipeline escalates deploy issues as tickets | AD-ACAP → AMS-IP | Class B MCP (ITSM) |
| AD learns from production patterns | AMS-PIS / IMS-PIS → KL → AD-ACAP | Shared KL read |
| AMS escalates infra-level incidents | AMS-ARP → IMS-AOP | A2A |
| IMS creates AMS tickets when manual help needed | IMS-AOP → AMS-IP | Class B MCP (ITSM) |
| All platforms write outcomes to KL | All → KL Ingest MCP | Class B MCP |
| All platforms governance-gate AI outputs | All → Governance Sidecar | Class B MCP |
| All platforms emit AI telemetry | All → AgentOps + AgentCore Observability | OTel GenAI Conventions |
| All platforms emit token cost | All LLM Gateways → IMS-FAG | Class B MCP / direct ingest |

These are the only sanctioned cross-RA flows. Any additional touchpoint requires programme-level review.

### 15.8 Shared services consumed by all three service lines

| Shared service | Purpose | Owner |
|---|---|---|
| **Knowledge Layer (KL)** | Hybrid retrieval substrate + optional KG | Dedicated KL owner |
| **Governance Sidecar** | Runtime governance gating | Governance authority |
| **Audit Log** | Compliance evidence and forensics | Governance authority |
| **MCP Catalog + Approval + Gateway + Identity + Health** | Integration plane | Platform team |
| **AgentOps Platform** | ADLC enforcement, evaluation, prompt management, drift, replay | AgentOps team |
| **Data Contracts Registry** | One contract per data product | Data architecture team |
| **Observability stack (four domains)** | Infra + Application + AI + Data observability | Platform team + AgentOps team + Data team |
| **Identity Provider** | IAM Identity Center + AgentCore Identity | Security |

---

## 16. Resolved decisions log

All 40 questions raised during pillar and platform design have been triaged and resolved. This log preserves the decisions for audit. Items marked **DECIDED** were resolved on architectural grounds; items marked **DEFAULT TAKEN** had a recommended default position locked in pending later confirmation if jurisdictional or commercial circumstances change.

### 16.1 Pillar boundaries

| # | Question | Decision |
|---|---|---|
| Q1 | "Change & Problem Process Mining" — AMS-01 or AMS-03? | **DECIDED:** AMS-03 (operational analytics, feeds triage/routing/workflow redesign) |
| Q2 | Split change/problem out of AMS-05? | **DECIDED:** No — same architectural pattern as resolution (11-node graph, OPA gates, ITSM MCP path) |
| Q3 | Compliance boundary AMS-06 vs IMS-08? | **DECIDED:** AMS-06 owns app-layer compliance; IMS-08 owns infra-layer; IMS-07 owns FinOps compliance |
| Q4 | "Intent-to-Roadmap" — AD-01 or AD-07? | **DECIDED:** AD-01 (planning capability, not agent-improvement loop) |
| Q5 | Design patterns + tech debt — AD-02 or AD-03? | **DECIDED:** AD-02 (architecture-level concerns); AD-03 stays narrowly focused on code production |
| Q6 | Split Legacy Code Refactoring from AD-03? | **DECIDED:** No — single agent capability at narrowed scope |
| Q7 | Security Architecture (Foundation) — IMS-01 or IMS-08? | **DECIDED:** IMS-08 (security team owns from threat-modelling onward) |
| Q8 | Split IMS-03 into Provisioning/Patch/Drift? | **DECIDED:** No — same architectural concerns; pillar split adds bureaucracy without value |
| Q9 | IMS-05 with 16 use cases — split? | **DECIDED:** Yes — IMS-05a (signal-driven auto-remediation) + IMS-05b (workflow-driven infra ops); both on IMS-AOP |
| Q10 | IMS-06 — real pillar or misc bucket? | **DECIDED:** Real pillar, renamed *Endpoint & Specialised Workload Operations* to be explicit |
| Q11 | Does IMS-07 fold into IMS-08? | **DECIDED:** No — different audiences, cadences, toolchains |

### 16.2 Use-case mapping

| # | Question | Decision |
|---|---|---|
| Q12 | Tech Debt Analysis — IMS-04 or IMS-07? | **DECIDED:** IMS-07 (asset/audit activity, not forecast) |
| Q13 | "Predictive Reliability & FinOps Autonomy" — IMS-05 or IMS-07? | **DECIDED:** Split — Predictive Reliability → IMS-05a; FinOps Autonomy → IMS-07 |
| Q14 | RCA Gap Detection — AMS-04 or AMS-07? | **DECIDED:** AMS-04 (pattern analytic, not improvement loop) |
| Q15 | SLA Monitoring — IMS-02 or IMS-07? | **DECIDED:** Split — real-time SLA Monitoring stays IMS-02; SLA Reporting & Credit Analysis → IMS-07 |

### 16.3 Cross-cutting

| # | Question | Decision |
|---|---|---|
| Q16 | Sprint Planning Ph 0 → Ph 3 in AD-05 — too wide? | **DECIDED:** Keep — maturity arc *is* the point; same data sources and downstream consumers throughout |

### 16.4 Platform count

| # | Question | Decision |
|---|---|---|
| Q17 | AMS-CMP — separate or fold into AMS-IP? | **DECIDED:** Keep separate (failure-domain isolation; read vs write degradation differs) |
| Q18 | IMS at 6 platforms — collapse? | **DECIDED:** No collapse; each platform has distinct cadence/audience/toolchain |
| Q19 | AD-DWP — "platform" the right word? | **DECIDED:** Renamed to "Developer Workspace" in prose; AD-DWP abbreviation retained for cross-references |

### 16.5 Boundary edge cases

| # | Question | Decision |
|---|---|---|
| Q20 | AMS-04 split across AMS-IP and AMS-PIS — acceptable? | **DECIDED:** Acceptable; documented as deliberate latency-driven split |
| Q21 | AD-01..04 split across DWP and ACAP — acceptable? | **DECIDED:** Acceptable; this is the AD hybrid pattern (§5.4), the core architecture |
| Q22 | IMS-06 specialised ops — dedicated platform? | **DECIDED:** No; same agentic execution pattern as endpoint mgmt; pillar-level distinction sufficient |

### 16.6 AD-specific (Kiro, AWS, identity)

| # | Question | Decision |
|---|---|---|
| Q23 | CodeCatalyst vs GitHub Actions? | **DECIDED:** GitHub Actions on AWS runners (D12) |
| Q24 | GitHub Enterprise on AWS for code hosting? | **DECIDED:** Yes (D12) |
| Q25 | Single-region initial, multi-region Ph 4+? | **DECIDED:** Yes (D22) |
| Q26 | Powers governance approval process? | **DECIDED:** Yes — AD-IP hosts Powers Approval Workflow on same Step Functions infra as MCP Approval |
| Q27 | Baseline AGENTS.md / coding standards centrally distributed? | **DECIDED:** Yes — AD-IP hosts Steering File Distribution |
| Q28 | Kiro version pinning? | **DECIDED:** Pinned, coordinated quarterly upgrade (D23) |
| Q29 | Developer identity in audit log — privacy framework? | **DEFAULT TAKEN:** Pseudonymisation with two-person re-id (D19); per-jurisdiction tuning during rollout |
| Q30 | Offline tolerance — read vs write paths? | **DEFAULT TAKEN:** Reads + suggestions offline; writes require connectivity (D20) |
| Q31 | AD → IMS provisioning authority? | **DECIDED:** Approval gate Ph 1-3; autonomous Ph 4+ (matches §11.5 maturity-staged autonomy) |
| Q32 | Multi-AZ from Ph 1? | **DECIDED:** Yes (§10.2) |

### 16.7 Tooling (sizing-blocking)

| # | Question | Decision |
|---|---|---|
| Q33 | Knowledge graph — Neo4j vs Neptune vs none? | **DEFAULT TAKEN:** Deferred to Phase 3 (D14); add only if hybrid retrieval hits quality ceilings |
| Q34 | AgentOps platform shape? | **DEFAULT TAKEN:** Managed mix — AgentCore Observability + Bedrock Evaluations + Langfuse self-hosted (D16) |
| Q35 | Constitutional SDD — build on Kiro vs commercial? | **DEFAULT TAKEN:** Build on Kiro steering files + ADLC decision logs (D21) |
| Q36 | Reranker — Cohere on Bedrock vs self-hosted? | **DEFAULT TAKEN:** Cohere Rerank on Bedrock (D15) |
| Q37 | Hybrid retrieval — single store (OpenSearch k-NN) vs two stores? | **DEFAULT TAKEN:** pgvector + OpenSearch (D13) — co-location with RDS + BM25 quality |
| Q38 | Data observability — AWS-native vs commercial? | **DEFAULT TAKEN:** AWS-native — Glue Data Quality + OpenLineage on Marquez (D18) |
| Q39 | AI observability — Langfuse vs Datadog vs AgentCore alone? | **DEFAULT TAKEN:** Langfuse self-hosted + AgentCore Observability + Bedrock Evaluations (D16) |
| Q40 | Memory framework — AgentCore Memory vs Mem0/Letta? | **DEFAULT TAKEN:** AgentCore Memory (D17) |

---

## 17. Next step — component decomposition

With the architecture locked, the next step is **component-level decomposition** for each of the 13 platforms — internal component lists, message flows, external integrations (MCP/A2A), and reference tech per component. This output feeds directly into the C4 L2 diagram.

For each platform we will produce:
- Component list with role, latency budget, ownership
- Internal data flows (between components within the platform)
- External integrations (Class B/C MCP servers called, A2A endpoints exposed)
- Reference tech (from the locked D-decisions, no further choice)
- Failure modes and degradation behaviour per component (per §10.4)

The use-case workbook will also be updated to v5 to reflect the use-case-level splits from Q13 and Q15.

---

## 18. Per-deployment trade-offs to re-surface

§16 logs the trade-offs **already resolved for this programme**. The following are trade-offs that should be **re-examined for each new client deployment** under this reference architecture, because the right answer is genuinely client-specific (risk appetite, scale, regulatory environment, existing stack). Defaults shown reflect this programme's locked positions where one exists; clients with different shapes will diverge.

| Trade-off | Option A | Option B | Programme default |
|---|---|---|---|
| Platform decomposition (AMS/IMS) | Three-platform (IP / PIS / ARP-class) | Single-platform with internal modules | Three-platform (§15.0) |
| Knowledge layer scope | Shared cross-line (federated) | Per-line local stores | Shared (D3) |
| Risk Scorer output | Normalised score (consumer thresholds) | Binary allow/deny decision | Normalised score with thresholds at consumer |
| Agent orchestration | LangGraph multi-step graphs | FastAPI deterministic flows | LangGraph (D5) |
| Governance placement | Synchronous critical-path gate | Async with retraction window | Synchronous (§12.3, P8) |
| Write strategy to ITSM | Single PATCH per cycle | Per-field writes | Single PATCH per cycle |
| Autonomy progression | Evidence-based with trust-decay | Policy-capped at fixed tier | Evidence-based with trust-decay (§1.6) |
| Time-series ingest | Dual (separate infra & APM) | Single unified | Dual hypertables on TimescaleDB (§15.4 IMS-IP) |
| LLM access | Single shared wrapper across lines | Per-line LLM Gateway | Per-line Gateway (D4) |
| Action vocabulary (agent → tools) | Controlled list (curated MCP catalog) | Free-form LLM-generated | Controlled list (§8.2 MCP Catalog) |
| Memory framework | Managed (AgentCore Memory) | OSS (Mem0 / Letta self-hosted) | AgentCore Memory (D17) |
| Knowledge graph | Deploy from Phase 1 | Defer; add only on retrieval-quality ceiling | Defer to Phase 3 (D14) |
| AI observability | Single vendor (e.g., Datadog) | Composed stack (Langfuse + AgentCore + Bedrock Evals) | Composed stack (D16) |
| Developer-identity attribution (AD only) | Raw username/email | Pseudonymised with two-person re-id | Pseudonymised (D19) |
| Multi-region | From Phase 1 | Phase 4+ when justified | Phase 4+ (D22) |
| Cloud strategy (largest trade-off; full mapping in §19) | AWS-native / Azure-native | Platform-independent (Kubernetes + OSS) | AWS-native (D2) |

Document the choice and the rationale per deployment. A deviation from a programme default is not a defect; an undocumented deviation is.

---

## 19. Cloud strategy: AWS, Azure, platform-independent

This section is the canonical mapping that supports per-deployment cloud-strategy selection (referenced from P3, P4, D2, and §18). The architectural concerns in §§2–14 are cloud-neutral by design; this section maps each cloud-coupled component to its equivalent under three strategies. **Choose one strategy per deployment;** hybrid and multi-cloud variants are covered in §19.4.

### 19.1 Three strategies, when each fits

| Strategy | When it fits | When it doesn't |
|---|---|---|
| **AWS-native** | Existing AWS footprint; AgentCore + Bedrock alignment desired; managed-first preference; Linux Foundation MCP/A2A spec alignment | Client mandates Azure or sovereignty; Microsoft-shop with Entra ID as identity source of truth |
| **Azure-native** | Existing Azure / Microsoft 365 / Entra ID footprint; Foundry Agent Service alignment; Copilot Studio integration needs; clients standardised on Azure OpenAI | No Azure footprint and no Microsoft 365 leverage; Anthropic Claude as the strongly preferred foundation model and no need to proxy via Foundry |
| **Platform-independent** | Multi-cloud or hybrid-cloud mandate; sovereignty / on-prem / air-gapped requirements; strong OSS-first culture; aversion to managed-runtime lock-in; existing strong Kubernetes platform team | No Kubernetes platform team; client wants managed substrate to minimise ops burden; small scale where managed-first total cost is lower |

The choice is **per-deployment, not per-programme**. A consultancy can run an AWS deployment for client A, Azure for client B, and platform-independent for client C from the same reference architecture. The 11 invariants in §19.2 hold across all three.

### 19.2 Invariants — what does NOT change across strategies

These hold under every strategy and are non-negotiable. They are why §§2–14 of this document remain valid regardless of cloud choice.

| # | Invariant | Why |
|---|---|---|
| I1 | **LangGraph** as orchestration framework | D5; OSS; runs on any compute substrate |
| I2 | **OPA + Rego** at the Governance Sidecar | D9; OSS; deploys anywhere |
| I3 | **OpenTelemetry GenAI Semantic Conventions** for AI telemetry | D6, P6; open standard; backends are interchangeable |
| I4 | **OpenLineage** for data lineage | P6; open standard |
| I5 | **FOCUS** schema for billing normalisation | P6, §14.2; cross-cloud by design |
| I6 | **MCP** as agent-to-tool protocol | P5; open spec; supported on AWS APIGW, Azure APIM, and OSS gateways |
| I7 | **A2A** as agent-to-agent protocol | D7, P5; open spec; agnostic transport |
| I8 | **The 11-node resolution graph** and ADLC lifecycle | §6.4, §12.2; logical concerns, not infrastructure |
| I9 | **Hybrid retrieval pattern** (dense + sparse + reranker) | §4.3; the stores swap, the pattern doesn't |
| I10 | **Five memory tiers** (working through procedural) | §7.2; logical model |
| I11 | **All §§9–14 architectural concerns** (observability, reliability, security, governance, AgentOps, FinOps) | The concerns are stable; the tools that implement them are what §19.3 maps |

### 19.3 Capability-to-tech mapping across strategies

For each capability used in this RA, the cloud-strategy options. The "Platform-independent" column is the OSS + Kubernetes path; assume Kubernetes (EKS / AKS / GKE / on-prem) as the compute substrate and that mature OSS components are deployed as Helm charts.

#### Compute & runtime (§2)

| Capability | AWS-native | Azure-native | Platform-independent |
|---|---|---|---|
| Container compute | EKS | AKS | Kubernetes (CNCF distro of choice) |
| Managed agent runtime | AgentCore Runtime | Foundry Agent Service (with bring-your-own-framework support for LangGraph) | LangGraph Platform self-hosted, or LangGraph on Kubernetes with custom session/identity layer |
| Serverless event compute | Lambda + Step Functions | Azure Functions + Durable Functions / Logic Apps | Knative + Argo Workflows |
| ML compute | SageMaker | Azure Machine Learning | Kubeflow + MLflow |
| Streaming bus | MSK (Kafka) | Event Hubs (Kafka-compatible) or Confluent on Azure | Apache Kafka or Redpanda on Kubernetes |
| Developer-laptop AI IDE | Kiro (AWS) | VS Code + GitHub Copilot + Foundry SDK; or Kiro (Kiro is editor-agnostic, AWS-hosted backend remains) | Continue.dev / Cline / Aider with local or BYO model |

**Gap note:** Foundry Agent Service in 2026 supports BYO frameworks (LangGraph, Microsoft Agent Framework, LangChain, CrewAI, LlamaIndex) on the Azure side, which preserves D5 (LangGraph as the brain). Platform-independent has no equivalent of a managed *agent* runtime — you assemble it from LangGraph + Kubernetes + your own session/checkpoint/identity layer. This is the largest ops-burden delta of the three strategies.

#### Data architecture (§3)

| Capability | AWS-native | Azure-native | Platform-independent |
|---|---|---|---|
| Relational (OLTP) | RDS PostgreSQL | Azure Database for PostgreSQL Flexible Server | PostgreSQL on Kubernetes (CloudNativePG operator) |
| Time-series | TimescaleDB on RDS, CloudWatch Metrics | TimescaleDB on Azure PostgreSQL, Azure Monitor metrics | TimescaleDB on Kubernetes, Prometheus + Thanos for metrics |
| Vector store (dense) | pgvector (HNSW) on RDS | pgvector on Azure PostgreSQL Flexible Server (DiskANN also available, Azure-specific) | pgvector on PostgreSQL, or Qdrant / Milvus / Weaviate on Kubernetes |
| Lexical store (sparse, BM25) | OpenSearch Service | Azure AI Search (native hybrid: vector + BM25 + semantic ranker in one product) | OpenSearch OSS or Elasticsearch OSS on Kubernetes |
| Document / blob | S3 + Glacier | Azure Blob Storage (Hot/Cool/Archive tiers) | MinIO on Kubernetes; or any S3-compatible object store |
| Cache | ElastiCache Redis | Azure Cache for Redis / Azure Managed Redis | Redis Operator on Kubernetes; or DragonflyDB |
| Stream / event log | MSK Kafka | Event Hubs (Kafka API) | Apache Kafka or Redpanda on Kubernetes |
| Metadata catalogues | DynamoDB | Azure Cosmos DB for NoSQL | etcd, FoundationDB, or PostgreSQL JSONB |

**Strategic note for Azure-native:** Azure AI Search natively provides hybrid retrieval (BM25 + vector + semantic ranker) in a single product. This means an Azure deployment can **collapse D13 from two stores (pgvector + OpenSearch) to one store**, with the reranker built in (potentially making D15 — Cohere reranker — optional). This is a real simplification and a Q37 outcome that would differ from AWS-native.

#### Knowledge architecture (§4)

| Capability | AWS-native | Azure-native | Platform-independent |
|---|---|---|---|
| Embedding model | Bedrock-hosted (Titan, Cohere Embed) | Azure OpenAI text-embedding-3-large / -small | Self-hosted (e.g., `bge-large-en-v1.5`, `nomic-embed-text`) on a model-serving stack (vLLM, TGI, Ollama) |
| Reranker | Cohere Rerank on Bedrock (D15) | Azure AI Search semantic ranker (built-in) or Cohere Rerank via Azure Marketplace | Cohere Rerank model self-hosted, or `bge-reranker-large` on vLLM |
| Knowledge graph (deferred per D14) | Neptune or Neo4j on EKS | Cosmos DB Gremlin API, or Neo4j on AKS | Neo4j Community / Memgraph on Kubernetes |
| RAG evaluation harness | RAGAS (OSS); Bedrock Evaluations for batch | RAGAS (OSS); Foundry Evaluations | RAGAS (OSS); custom harness in Kubernetes |

#### Foundation models (§8.4)

| Capability | AWS-native | Azure-native | Platform-independent |
|---|---|---|---|
| Primary reasoning model | Claude Sonnet on Bedrock | GPT-4-class on Azure OpenAI; Claude available via Azure Marketplace; Foundry catalogue includes Mistral, Llama, etc. | Self-hosted via vLLM / TGI / Ollama (Llama, Qwen, Mistral, DeepSeek); or BYO API key to any provider |
| Cost-efficient throughput model | Amazon Nova | GPT-4o-mini, Phi-3.5; or any small model in Foundry catalogue | Smaller self-hosted (Llama 3 8B, Phi-3, Qwen 7B) |
| Model plane | Bedrock | Azure OpenAI + Foundry catalogue | Direct provider APIs proxied through LiteLLM, or self-hosted models behind a unified gateway |
| Multi-model routing & cost controls | Custom on Bedrock Cross-Region Inference (D8) | Azure APIM AI Gateway (built-in routing, semantic cache, token rate-limiting, FinOps integration) | LiteLLM Proxy, Bifrost, or Kong AI Gateway |

**Strategic note for Azure-native:** Azure APIM AI Gateway covers what AWS splits across the LLM Gateway (§8.4) and parts of the MCP Gateway (§8.3) in a **single product**. This may simplify §8.3+§8.4 from "two gateways" to "one gateway" on Azure, but at the cost of a larger blast radius if APIM is misconfigured.

#### Integration plane (§8)

| Capability | AWS-native | Azure-native | Platform-independent |
|---|---|---|---|
| MCP Gateway | AgentCore Gateway + API Gateway + Lambda | Azure APIM AI Gateway (native MCP support since 2025) | Kong AI Gateway, Bifrost, or Envoy-based custom; with WASM plugins for OPA |
| MCP Catalog & approval | DynamoDB + Step Functions UI | Azure API Center + Logic Apps approval | PostgreSQL-backed registry + Argo Workflows for approval |
| Outbound OAuth / identity broker | AgentCore Identity + IAM Identity Center + STS | Entra Agent ID + Workload Identity Federation | Keycloak + SPIFFE/SPIRE for workload identity |
| Egress DLP | AWS Network Firewall + custom Lambda | Azure Firewall + APIM content filters | OPA + custom Envoy filter, or commercial DLP via webhook |

#### Identity (§11)

| Capability | AWS-native | Azure-native | Platform-independent |
|---|---|---|---|
| Human identity provider | IAM Identity Center (federated to corporate IdP) | Microsoft Entra ID (native) | Keycloak, Authentik, or Dex (OIDC federation to corporate IdP) |
| Agent identity | AgentCore Identity (per-agent scoped IAM roles) | Entra Agent ID (purpose-built agent identity objects with blueprints, sponsor, RBAC; GA 2026) | SPIFFE / SPIRE workload identity + per-agent OIDC identities issued by Keycloak |
| Secrets | AWS Secrets Manager | Azure Key Vault | HashiCorp Vault on Kubernetes; or External Secrets Operator |
| Network | VPC + PrivateLink | VNet + Private Endpoint | Cilium / Istio mTLS + NetworkPolicy |

**Strategic note for Azure-native:** Entra Agent ID introduces purpose-built identity *objects* for agents (agent identity, agent identity blueprint, sponsor role, agent user) that AWS's IAM-role-per-agent model doesn't have. For organisations with M365 / Entra leverage, this is a real authority-and-accountability win — the *sponsor* concept maps directly onto §12.2 ADLC ownership.

#### Observability (§9)

| Capability | AWS-native | Azure-native | Platform-independent |
|---|---|---|---|
| Infrastructure + Application telemetry | CloudWatch + X-Ray + Application Signals | Azure Monitor + Application Insights | Prometheus + Loki + Tempo + Grafana (LGTM stack) |
| AI telemetry (OTel GenAI per D6) | AgentCore Observability emits OTel GenAI; Langfuse for prompt/trace deep-dive (D16) | Application Insights "Agents" view (OTel GenAI native); Foundry Observability dashboard; Langfuse self-hosted retained for prompt registry | OTel Collector → Langfuse (self-hosted) + Grafana Tempo for trace correlation |
| Eval pipeline | Bedrock Evaluations (batch) + Galileo Luna + Langfuse | Foundry Evaluations + Galileo Luna + Langfuse | RAGAS + Galileo Luna (OSS edition) + Langfuse |
| Data observability (§9.4) | Glue Data Quality + OpenLineage on Marquez (D18) | Microsoft Purview Data Quality + OpenLineage on Marquez | Great Expectations + OpenLineage on Marquez |
| Lineage | OpenLineage (open) → Marquez | OpenLineage (open) → Marquez or Purview Lineage | OpenLineage → Marquez |

**Critical:** D6 (OTel GenAI Semantic Conventions) is what makes AI observability portable across all three strategies. Azure Application Insights, AWS CloudWatch, and Grafana Tempo all consume the same OTel GenAI spans. The dashboards differ; the data does not.

#### Security tooling (§11)

| Capability | AWS-native | Azure-native | Platform-independent |
|---|---|---|---|
| Cloud-native vulnerability mgmt | Inspector | Microsoft Defender for Cloud | Trivy + Falco on Kubernetes |
| Detection & response | Security Hub + GuardDuty | Microsoft Sentinel + Defender for Cloud | Wazuh + Falco; OSS SIEM (e.g., OpenSearch SIEM, Elastic SIEM OSS) |
| KMS | AWS KMS | Azure Key Vault Managed HSM | HashiCorp Vault Transit, or OpenBao |
| Container signing | AWS Signer | Azure Container Registry signing (Notary v2) | Cosign / Sigstore |

#### Source control & CI/CD (§12, D12)

| Capability | AWS-native | Azure-native | Platform-independent |
|---|---|---|---|
| Source control | GitHub Enterprise on AWS (D12) | Azure DevOps Repos, or GitHub Enterprise on Azure | Gitea / Forgejo, or self-hosted GitLab |
| CI/CD | GitHub Actions on AWS runners (D12) | Azure DevOps Pipelines or GitHub Actions on Azure runners | Argo CD + Tekton, or GitLab CI |

### 19.3a Decisions that change by strategy

For per-deployment review when selecting a non-default strategy:

| Decision | AWS-native (locked default) | Azure-native | Platform-independent |
|---|---|---|---|
| **D5** Agent substrate | AgentCore Runtime + LangGraph | Foundry Agent Service + LangGraph (BYO framework) | Kubernetes + LangGraph + custom session/identity layer |
| **D8** Model plane | Bedrock + Cross-Region Inference | Azure OpenAI + Foundry catalogue + APIM AI Gateway | LiteLLM Proxy or self-hosted model serving (vLLM / TGI) |
| **D13** Hybrid retrieval | pgvector + OpenSearch (two stores) | Azure AI Search (single store, hybrid built-in) **or** pgvector + Azure AI Search | pgvector + OpenSearch OSS, or Qdrant + OpenSearch |
| **D15** Reranker | Cohere Rerank on Bedrock | Azure AI Search semantic ranker (built-in, no separate reranker call) | `bge-reranker-large` self-hosted on vLLM, or Cohere via direct API |
| **D16** AI observability | AgentCore Observability + Langfuse + Bedrock Evals | App Insights Agents view + Foundry Evaluations + Langfuse | OTel Collector + Langfuse + RAGAS + Galileo Luna OSS |
| **D17** Memory framework | AgentCore Memory | Foundry agent memory (managed) or Mem0/Letta self-hosted | Mem0 or Letta self-hosted; or PostgreSQL-backed custom |
| **D18** Data observability | Glue Data Quality + OpenLineage on Marquez | Purview Data Quality + OpenLineage on Marquez | Great Expectations + OpenLineage on Marquez |
| **D12** Code hosting | GitHub Enterprise on AWS + GitHub Actions | Azure Repos / GitHub on Azure + Azure Pipelines / Actions | Gitea or self-hosted GitLab + Argo CD / Tekton |

D1, D3, D4, D6, D7, D9, D10, D11, D14, D19, D20, D21, D22, D23 do **not** change across strategies.

### 19.4 Hybrid and multi-cloud notes

Few real deployments are pure. Common hybrid shapes seen in practice:

| Hybrid shape | Typical reason | Implications |
|---|---|---|
| **AWS compute + Entra ID identity** | Microsoft-shop with strong identity governance, but AWS for AI workloads | Identity propagation (P9) needs Entra ↔ IAM Identity Center federation; the OPA Sidecar consumes the federated identity. Workable, with one extra hop in the identity chain. |
| **Azure compute + Bedrock for Claude** | Strong preference for Claude as the reasoning model, but Azure footprint everywhere else | Foundation-model traffic egresses Azure; cost-attribution dimension (§14.3) needs AWS billing pipe in parallel to Azure billing. FOCUS schema (I5) makes this clean. |
| **Platform-independent compute, but managed AI services** | OSS-first culture for compute, but managed reranker/embedding/eval to reduce ops burden | Class C MCP egress to AWS Bedrock or Azure OpenAI; OPA sidecar enforces egress allow-list. The MCP Gateway is what makes this clean — the agent doesn't know the model is off-cluster. |
| **Active-active across AWS + Azure for sovereignty** | Regulator demands two-cloud resilience; or supplier-risk policy | D22 multi-region becomes multi-cloud; KL replication is the hardest part (embeddings + lexical indexes must stay synchronised across clouds). Treat as Phase 4+ scope; do not attempt Phase 1. |

**General rule:** the integration plane (§8) is what makes hybrid work. MCP Gateway + LLM Gateway + Governance Sidecar can route to whichever cloud hosts the target, as long as identity (P9) and trace context (P10) propagate cleanly. **The agent does not know which cloud answers** — that is the whole point of having a centralised integration plane.

### 19.5 Selecting a strategy — quick decision aid

A simple decision sequence:

1. **Is there a regulatory or sovereignty constraint mandating a specific cloud or on-prem?** If yes, that constraint is the answer. Skip ahead.
2. **Is the client already 60%+ on one of AWS or Azure?** If yes, default to that cloud. The cost of cross-cloud is rarely worth fighting an existing footprint.
3. **Is the client Microsoft-365-heavy and using Entra ID as the corporate identity source?** If yes, Azure-native gains real identity-governance leverage (Entra Agent ID, sponsor model). Worth +1 point for Azure.
4. **Does the client have a strong Kubernetes platform team and an OSS-first culture?** If yes, platform-independent becomes viable. Without that team, do not pick it.
5. **Is Anthropic Claude strongly preferred and the rest of the stack neutral?** If yes, AWS-native is the lowest-friction path (Bedrock first-party Claude). Azure-native is also viable via Marketplace; platform-independent via Claude API direct.
6. **Default fallback when nothing dominates:** AWS-native (D2). This is the locked programme default not because AWS is universally best but because it is the strategy that this RA has been pressure-tested against.

Document the strategy choice **and** the rationale on Day 1 of an engagement. Strategy switches mid-engagement are expensive; they are not impossible, but they reset Phase 1.

---

## 20. High-level task list for delivery estimation

This section enumerates the high-level delivery tasks implied by the architecture. **It is a parent reference**, not the estimation itself — each task here is the source for one or more concrete estimation records produced downstream (in Jira / Azure DevOps / xlsx). Decompose each high-level task into delivery-team-sized tickets at estimation time; do not estimate directly off this list.

### 20.1 Conventions

| Field | Values | Meaning |
|---|---|---|
| **Size** | S / M / L / XL | Rough effort for a 3–6-engineer team: S ≤ 4 weeks; M 4–10 weeks; L 10–20 weeks; XL 20+ weeks |
| **Complexity** | Low / Med / High | Technical novelty, integration breadth, governance scrutiny, regression-test surface |
| **Phase** | 1 / 2 / 3 / 4 | Programme phase from §15.5 — Trust foundation / Orchestration / Scale / Autonomy |
| **Cloud-coupled** | Yes / No | "Yes" means the work content varies per §19 strategy choice; "No" means cloud-neutral |
| **Track** | A / B / C | A = Foundational shared services; B = Per-service-line platforms; C = Cross-cutting continuous |
| **Refs** | §-references | Section(s) of this document that define the task scope |

**Sizing baseline:** assumes the locked AWS-native strategy (D2). Under §19 alternative strategies, cloud-coupled tasks may move ±1 size due to different managed-vs-self-host trade-offs. Re-size at strategy commit.

**Tasks excluded by design:**

- Continuous Learning per service line (AMS-07, AD-07, IMS-09) — D11 platform-tax against the shared AgentOps platform, not a separate delivery
- Architecture documentation — the architecture exists; tasks below are delivery
- Discovery & assessment — pre-engagement work, sized separately

### 20.2 Track A — Foundational shared services

Built once at programme level (P2 / D3); serve all three service lines. These are the critical-path Phase 1 tasks; nothing in Track B can start without them.

| ID | Task | Size | Complexity | Phase | Cloud-coupled | Refs |
|---|---|---|---|---|---|---|
| **A01** | **Identity foundation** — Set up IdP federation, scoped agent identities, AgentCore Identity / Entra Agent ID / SPIFFE-SPIRE (per §19); short-lived credentials; outbound OAuth | M | High | 1 | Yes | §11.3, §8.4, P9 |
| **A02** | **Network & security baseline** — VPC/VNet, private subnets, PrivateLink/Private Endpoint, TLS, KMS, Secrets Manager | M | Med | 1 | Yes | §11.3 |
| **A03** | **Data store foundation** — Relational (RDS/Azure PG/CNPG), time-series (TimescaleDB), object (S3/Blob/MinIO), cache (Redis), metadata (DynamoDB/Cosmos/etcd) | L | Med | 1 | Yes | §3.3 |
| **A04** | **Streaming bus** — MSK/Event Hubs/Kafka-on-K8s; Kafka Connect; DLT pattern; producer/consumer SDK | M | Med | 1 | Yes | §3.3, §3.6 |
| **A05** | **Data Contracts Registry** — Schema, owner, freshness SLA, access policy, lineage, classification, decommissioning; runtime cache + management UI | M | Med | 1 | No | §3.4 |
| **A06** | **Anchor data products** — Define and publish anchor entities (ticket, incident, CI, runbook, spec, repo, ADR, telemetry, alert, patch, cost_line, resolution_signature) with contracts | L | Med | 1–2 | No | §3.5 |
| **A07** | **Knowledge Layer — dense vector store** — pgvector on Postgres / Azure AI Search (per §19); HNSW indexing; partitioned per service line | M | Med | 1 | Yes | §4.2, D13 |
| **A08** | **Knowledge Layer — lexical store** — OpenSearch / Azure AI Search BM25 lane; index lifecycle; rebuild pipeline | M | Med | 1 | Yes | §4.2, D13 |
| **A09** | **Knowledge Layer — reranker integration** — Cohere on Bedrock / Azure AI Search semantic ranker / self-hosted bge-reranker (per §19) | S | Low | 2 | Yes | §4.2, D15 |
| **A10** | **Knowledge Layer — retrieval pipeline** — HyDE/query transformation; hybrid search merge; reranker; minimum-viable-data context assembly; published as LangGraph node library | M | High | 2 | No | §4.3 |
| **A11** | **Knowledge Layer — ingestion gateway & freshness** — KL Ingest MCP server (Class B); provenance capture; embedding-model version tracking; drift-triggered re-embedding | M | Med | 1–2 | No | §4.5 |
| **A12** | **MCP Catalog & Approval Workflow** — Registry on DynamoDB/Cosmos; approval pipeline (security review, data residency, tool-description poisoning scan, SBOM, owner assignment) | M | Med | 1 | Yes | §8.3 |
| **A13** | **MCP Gateway** — Egress proxy; OAuth 2.1; DLP; rate limiting; kill-switches; virtual MCP servers; W3C Trace Context propagation | L | High | 1–2 | Yes | §8.3 |
| **A14** | **LLM Gateway (per service line, same architecture)** — Multi-model routing; fallback chains; token-aware rate limits; cost-velocity circuit breakers; prompt compression; semantic cache; OTel GenAI emission; PII guard; determinism controls | L | High | 2 | Yes | §8.4, D4 |
| **A15** | **AgentCore Runtime / Foundry Agent Service / LangGraph-on-K8s** — Substrate for agent execution per §19; session lifecycle; identity propagation; trace propagation; checkpointer integration | L | High | 2 | Yes | §2.2, D5 |
| **A16** | **Governance Sidecar (OPA + Rego)** — Runtime gate; allow/deny/HITL; append-only audit-log writer; reproducibility metadata; default-deny on unavailable | L | High | 1 | No | §12.3, D9 |
| **A17** | **Audit Log** — Append-only, tamper-evident; RDS JSONB hot + S3 Glacier cold; 12-month hot retention; pseudonymised developer ID with two-person re-id (AD) | M | Med | 1 | No | §12.5, D19 |
| **A18** | **OTel Collector & infra/app observability** — CloudWatch / Application Insights / LGTM stack (per §19); X-Ray/Tempo traces; RED + USE metrics | M | Med | 1 | Yes | §9.2 |
| **A19** | **AI observability stack** — AgentCore Observability / App Insights Agents view / OTel Collector emitting GenAI spans (D6); Langfuse self-hosted; span-event prompt/response capture with PII filter | L | High | 1–2 | Yes | §9.3, D16 |
| **A20** | **Data observability** — Glue Data Quality / Purview / Great Expectations; OpenLineage on Marquez; embedding drift + retrieval-quality regression detectors; KL staleness | M | Med | 2 | Yes | §9.4, D18 |
| **A21** | **AgentOps platform — prompt registry** — Langfuse self-hosted; Git source-of-truth; versioned prompts as signed artefacts; A/B framework; prompt SBOM capture | M | Med | 1–2 | No | §13.4, D16 |
| **A22** | **AgentOps platform — eval pipeline** — Heuristic + Luna-style + LLM-as-judge + agent-as-a-judge + human annotation; RAGAS retrieval eval; cost-per-eval routing | L | High | 2 | No | §13.3 |
| **A23** | **AgentOps platform — Continuous Learning loop (D11)** — Outcome aggregation; drift detection; eval triggers; promotion-through-ADLC; per-service-line configurations on shared platform | L | High | 3 | No | §5.4, §13.5 |
| **A24** | **A2A protocol substrate** — Agent Card publishing at well-known endpoint; task lifecycle implementation; SSE + JSON-RPC 2.0 transport; trace context in message metadata | M | High | 2–3 | No | §6.6, D7 |
| **A25** | **Memory framework integration** — AgentCore Memory / Foundry memory / Mem0 (per §19) wired across all five tiers; cross-line federation rules; memory-write Sidecar gate | M | High | 2 | Yes | §7, D17 |
| **A26** | **FOCUS billing pipeline** — Cloud + SaaS + AI token cost ingestion; FOCUS schema normalisation; per-team / per-use-case / per-agent attribution; chargeback views | M | Med | 2 | No | §5.5, §14 |
| **A27** | **Continuous red-teaming machinery** — Synthetic adversarial inputs; multi-modal & MCP attack scenarios; RL-trained adversarial agents for high-stakes platforms; attack-success-rate drift gate | M | High | 3 | No | §11.4 |

**Track A total:** 27 high-level tasks. Critical-path subset for any Phase 1 start: A01, A02, A03, A04, A05, A12, A16, A17, A18 — without these nine, Track B cannot begin.

### 20.3 Track B — Per-service-line platforms

Per-service-line platforms (§15.2 / §15.3 / §15.4). Each platform depends on the relevant Track A foundations.

#### 20.3.1 AMS platforms (4)

| ID | Task | Size | Complexity | Phase | Cloud-coupled | Refs |
|---|---|---|---|---|---|---|
| **B-AMS-01** | **AMS-IP build** — Eight-layer IP decomposition (§15.0.1); L1 ingestion (ITSM/monitoring webhooks); L3 dedup, cascade correlation, CMDB enrichment, priority scoring, qualification gate; L4 RAG + Resolver Copilot; advisory agents on AgentCore | XL | High | 1–2 | Yes | §15.2, §15.0.1 |
| **B-AMS-02** | **AMS-PIS build** — PIS four-layer pattern; recurrence forecasting; pattern-library on RDS; push-notify protocol to AMS-IP; Prophet/LSTM on SageMaker/Azure ML | L | High | 2–3 | Yes | §5.3, §15.2 |
| **B-AMS-03** | **AMS-ARP build** — 11-node resolution graph (§6.4); ReAct/Plan-then-Execute/Reflexion strategies per action class; HITL checkpointing; five memory tiers; OPA gates on every write-path | XL | High | 2–3 | No | §6.4, §15.2 |
| **B-AMS-04** | **AMS-CMP build** — Four-domain observability with AI emphasis; LLM-as-judge sampling of AMS-ARP; drift detection; periodic red-team; SLO publishing | L | Med | 3 | No | §15.2 |

#### 20.3.2 AD platforms (3)

| ID | Task | Size | Complexity | Phase | Cloud-coupled | Refs |
|---|---|---|---|---|---|---|
| **B-AD-01** | **AD-DWP configuration & rollout** — Kiro IDE pinning (D23); steering-file distribution mechanism; Class A stdio MCPs (filesystem, local Git, test runner); short-term memory; offline-tolerance behaviour (D20) | L | High | 1–2 | Yes | §15.3, D20, D21, D23 |
| **B-AD-02** | **AD-ACAP build** — AgentCore Runtime + LangGraph for autonomous delivery, refactoring, QA orchestration; cross-RA A2A flows; portfolio-level analysis for AD-01/AD-02; outcome publisher to KL | XL | High | 2–3 | Yes | §15.3, §6.6 |
| **B-AD-03** | **AD-IP build (Internal Developer Platform)** — Five-plane IDP; AI Governance / IP scanning / explainability gating; Powers Approval Workflow on Step Functions; Steering File Distribution service; license scanner; pseudonymised audit-log writer | L | High | 2–3 | Yes | §15.3, §12.4, D19 |

#### 20.3.3 IMS platforms (6)

| ID | Task | Size | Complexity | Phase | Cloud-coupled | Refs |
|---|---|---|---|---|---|---|
| **B-IMS-01** | **IMS-IP build** — Eight-layer IP; multi-source telemetry ingestion (cloud, OS, app); dual TimescaleDB hypertables (infra + APM); signal & alert intelligence advisory agents | XL | High | 1–2 | Yes | §15.4, §15.0.1 |
| **B-IMS-02** | **IMS-PIS build** — PIS four-layer; time-series ML (Prophet, LSTM, cascade models); capacity & failure forecasting; pre-check task generation | L | High | 2–3 | Yes | §5.3, §15.4 |
| **B-IMS-03** | **IMS-CPP build (Control Plane)** — Plan-then-Execute orchestration; patch-wave management; A2A endpoint from AD; provisioning/config/drift workflows; governance gates per change-window | L | High | 2–3 | No | §15.4, §6.2 |
| **B-IMS-04** | **IMS-AOP build (Agentic Ops)** — 11-node graph; Reflexion for high-stakes prod-critical actions; supervisor + specialists topology; hosts IMS-05a (signal-driven), IMS-05b (workflow-driven), IMS-06 (endpoint & specialised) | XL | High | 2–3 | No | §15.4, §6.4 |
| **B-IMS-05** | **IMS-FAG build (FinOps & Asset Governance)** — FOCUS ingestion; AI token-cost aggregator across all RAs; tagging engine; invoice reconciliation; licensing automation; Inform→Optimise→Operate dashboards | L | Med | 2–3 | No | §14, §15.4 |
| **B-IMS-06** | **IMS-SPP build (Security Posture)** — Detection layer (prompt-injection/memory-poisoning/tool-misuse monitors); vulnerability scanning; governed response (Reflexion); audit layer; highest governance tier | L | High | 3 | No | §11, §15.4 |

**Track B total:** 13 high-level platform-build tasks. AMS-IP, AMS-ARP, AD-ACAP, IMS-IP, IMS-AOP are the five XL items — these dominate the critical path and should be the focus of risk-management attention.

### 20.4 Track C — Cross-cutting & continuous

| ID | Task | Size | Complexity | Phase | Cloud-coupled | Refs |
|---|---|---|---|---|---|---|
| **C01** | **ADLC roll-out** — Six-phase lifecycle operationalised; mandatory artefacts per phase; per-agent registration; behaviour-spec discipline; pre-release approval workflow | M | Med | 1–2 | No | §12.2 |
| **C02** | **Per-domain OPA policy authoring** — Rego policies per Domain (not per Segment, see Appendix B); allow/deny/HITL rules per action class; kill-switch policies; trust-decay rules at L4+ | L | High | Continuous from 1 | No | §12.3, §1.6, Appendix B |
| **C03** | **Cross-RA flow implementation** — A2A endpoints AD-ACAP→IMS-CPP, AMS-ARP→IMS-AOP; ITSM MCP for AD/IMS→AMS; KL outcome flow from all platforms | M | High | 2–3 | No | §15.7 |
| **C04** | **MCP server onboarding (ongoing)** — Per-tool review, virtual-MCP composition per agent class, identity scoping, kill-switch configuration | M | Med | Continuous from 2 | No | §8.2, §8.3 |
| **C05** | **Prompt CI/CD operationalisation** — Golden eval sets per agent; A/B framework live; prompt-SBOM at build; eval-gated promotion | M | Med | 2 | No | §13.4 |
| **C06** | **Autonomy tier progression governance** — Per-domain SLI baselines; 8-week evidence windows; trust-decay automation; tier-cap policies per regulated workload | M | High | 3–4 | No | §1.6, §11.5 |
| **C07** | **Disaster recovery validation** — RTO/RPO drills per asset class; audit-log read-replica failover; KL rebuild from sources; cross-region (Phase 4+ per D22) | M | Med | 2 → 4 | Yes | §10.5 |
| **C08** | **Multi-AZ from Phase 1** — RDS Multi-AZ; EKS/AKS node groups across AZs; MSK/Event Hubs multi-AZ brokers; health checks and failover testing | M | Med | 1 | Yes | §10.2 |
| **C09** | **Multi-region rollout** — D22; KL replication across regions is the hard part; FOCUS billing remains single-pipe; identity federation across regions | XL | High | 4+ | Yes | §10.2, D22 |
| **C10** | **FinOps Inform → Optimise → Operate operationalisation** — Tag inventory; chargeback/showback live; anomaly detection; budget enforcement via Governance Sidecar; forecast accuracy review | M | Med | 2–3 | No | §14.5 |
| **C11** | **Continuous red-team operation** — Embedded in ADLC Test & Release; per-release attack-success-rate drift gating; staging-environment adversarial agents; quarterly methodology review | M | High | 3 → continuous | No | §11.4 |
| **C12** | **Per-jurisdiction privacy tuning** — D19 default; works-council frameworks; per-jurisdiction overrides for developer-identity attribution; legal sign-off per rollout region | S | High | Per rollout | No | §12.5, D19 |
| **C13** | **Kiro version-pin upgrade cadence** — Quarterly coordinated upgrade per D23; regression eval suite; rollback plan; AGENTS.md baseline alignment | S | Low | Quarterly from 2 | No | D23 |
| **C14** | **Component-level decomposition output (§17)** — For each of the 13 platforms: component list, internal data flows, external integrations (MCP/A2A), reference tech, failure modes; feeds C4 L2 diagram | M | Med | 1 (one-time, blocks B-tasks) | No | §17 |
| **C15** | **Use-case workbook v5** — Reflect Q13 + Q15 splits; per-pillar count alignment; cross-link to platform delivery tasks | S | Low | 1 (one-time) | No | §15.6 |

**Track C total:** 15 cross-cutting / continuous tasks. C14 and C15 are explicit one-time gates that the next-step section (§17) already calls out and that **must complete before Track B builds begin** — they convert this architecture into the buildable component lists.

### 20.5 Summary

| Track | Tasks | XL | L | M | S |
|---|---:|---:|---:|---:|---:|
| A — Foundational shared | 27 | 0 | 9 | 17 | 1 |
| B — Per-service-line platforms | 13 | 5 | 8 | 0 | 0 |
| C — Cross-cutting & continuous | 15 | 1 | 1 | 10 | 3 |
| **Total** | **55** | **6** | **18** | **27** | **4** |

**XL tasks (six)** — these are the items most likely to slip and the natural focus of risk management:

- B-AMS-01 AMS-IP build
- B-AMS-03 AMS-ARP build
- B-AD-02 AD-ACAP build
- B-IMS-01 IMS-IP build
- B-IMS-04 IMS-AOP build
- C09 Multi-region rollout (Phase 4+)

**Critical-path Phase 1 set** (must finish before Phase 2 work begins): A01, A02, A03, A04, A05, A12, A16, A17, A18, C08, C14, C15.

**Highest-complexity tasks** regardless of size (where governance, security, novelty are stacked): A01, A13, A14, A15, A16, A19, A22, A23, A24, A25, A27, B-AMS-01, B-AMS-03, B-AD-01, B-AD-02, B-AD-03, B-IMS-01, B-IMS-03, B-IMS-04, B-IMS-06, C02, C06, C09, C11, C12.

### 20.6 How to consume this list downstream

For each task above, the estimation team should produce:
1. **Decomposition into 2–10 estimable sub-tasks** sized in person-days
2. **Acceptance criteria** referencing the validation criteria from §15 and the eval gates from §13
3. **Dependency map** to other tasks in this list (prerequisite / parallel-OK / hard-blocked)
4. **Risk register entry** for any task marked Complexity = High
5. **Owning team** assignment with named accountability
6. **Phase-gate alignment** — confirm the §15.5 phase assignment is right for the client's sequencing

The output of that downstream exercise — not this list — is what feeds into a delivery plan or commercial proposal.

---

## 21. License requirements

This section enumerates the licensing terms for every component named in this reference architecture, organised by the §19 capability categories. **Licenses change.** Three of the components below have changed license within the last 24 months (Elasticsearch 2024, Redis 2024 → 2025, MinIO archived Feb 2026). **Verify the current license at the vendor's site at the moment of procurement** — the table is a sighting baseline, not a contract.

### 21.1 What "license" means in this table

Three different concepts are conflated under the word "license"; the table distinguishes them.

| Concept | What it covers | Where it matters |
|---|---|---|
| **OSS license** | Source-code distribution, modification, copyleft contagion | Platform-independent strategy; self-hosted components in any strategy |
| **Service terms** | Usage of a managed cloud service under the cloud provider's master agreement | AWS-native, Azure-native managed services |
| **Subscription tier** | Which features require a paid tier in an otherwise-free product | GitHub Enterprise tier requirements; Langfuse Cloud vs OSS; Neo4j Community vs Enterprise |

A "tri-licensed" product (Redis 8.0, Elasticsearch 8.16+) means the licensee picks **one** license to comply with — typically the most permissive that suits the use case.

### 21.2 License risk categories

For decisions in §19.3a and procurement conversations, the architecture uses three risk categories:

| Risk | Definition | Examples |
|---|---|---|
| **Green** | Permissive OSS (Apache 2.0, MIT, BSD, MPL 2.0) — clear commercial use, no copyleft contagion | LangGraph, OPA, OpenTelemetry, OpenSearch, PostgreSQL/pgvector, Apache Kafka, Argo CD |
| **Amber** | Source-available or weak-copyleft (AGPL, ELv2, SSPL, BUSL, RSAL, Llama Community) — usable but with material restrictions | Redis 8.0, Elasticsearch, Terraform/Vault, Llama 4, MinIO, Grafana |
| **Red** | Closed managed service with vendor-specific terms, or unmaintained — requires service-agreement review or migration plan | Bedrock, Foundry Agent Service, Azure OpenAI, Cohere managed, MinIO Community Edition (archived 2026) |

Risk colour reflects **legal/compliance friction**, not quality. A Red component may be the right architectural choice — it just needs a documented service-agreement review.

### 21.3 Compute & runtime substrate

| Component | AWS-native | Azure-native | Platform-independent | License / terms | Risk |
|---|---|---|---|---|---|
| Container compute | EKS | AKS | Kubernetes | EKS/AKS — cloud service terms. Kubernetes — Apache 2.0 | Green/Red |
| Managed agent runtime | AgentCore Runtime | Foundry Agent Service | LangGraph + Kubernetes (custom session/identity layer) | AgentCore — AWS service terms. Foundry — Azure service terms. LangGraph — MIT (LangChain Inc.). LangGraph Platform self-hosted has a separate commercial tier | Red managed / Green OSS |
| Serverless event compute | Lambda + Step Functions | Azure Functions + Durable Functions | Knative + Argo Workflows | Cloud service terms (Lambda/Functions). Knative — Apache 2.0. Argo Workflows — Apache 2.0 | Green/Red |
| ML compute | SageMaker | Azure Machine Learning | Kubeflow + MLflow | Cloud service terms. Kubeflow — Apache 2.0. MLflow — Apache 2.0 | Green/Red |
| Streaming bus | MSK (managed Kafka) | Event Hubs (Kafka API) | Apache Kafka or Redpanda on K8s | Apache Kafka — Apache 2.0. Redpanda — **BSL 1.1** (source-available; converts to Apache 2.0 after 4 years) | Green / Amber Redpanda |
| Developer-laptop AI IDE | Kiro (AWS preview product) | VS Code + Copilot or Kiro | Continue.dev / Cline / Aider | Kiro — AWS service terms. VS Code — MIT (Microsoft). Copilot — separate commercial subscription. Continue.dev — Apache 2.0. Cline — Apache 2.0. Aider — Apache 2.0 | Mixed |

### 21.4 Data architecture

| Component | License / terms | Risk | Notes |
|---|---|---|---|
| RDS PostgreSQL / Azure Database for PostgreSQL | Service terms; underlying PostgreSQL is PostgreSQL License (permissive, BSD-style) | Red managed / Green OSS | — |
| pgvector extension | PostgreSQL License | Green | — |
| OpenSearch (OSS) | Apache 2.0 (OpenSearch Software Foundation, Linux Foundation since 2024) | Green | The Apache-licensed lineage after the Elasticsearch fork in 2021 |
| OpenSearch Service (AWS) | AWS service terms (over the Apache 2.0 codebase) | Red managed | — |
| Elasticsearch | Tri-licensed since Sept 2024: **AGPLv3** (OSI), **SSPL 1.0**, or **ELv2** — licensee picks one; binary distributions remain ELv2 | Amber | AGPLv3 option is OSI-approved; SSPL is *not* OSI-approved; ELv2 restricts offering as managed service |
| Azure AI Search | Azure service terms | Red managed | The native-hybrid product called out as a possible D13 simplification in §19.3 |
| TimescaleDB | Open-source community: **Apache 2.0**; advanced features (multi-node, compression policies, continuous aggregates beyond basic) — **Timescale License (TSL)**, source-available with restrictions on offering as competing managed service | Amber | Self-hosted on K8s requires choosing which features are TSL-licensed; managed Timescale Cloud uses standard service terms |
| Redis 8.0+ | Tri-licensed since May 2025: **AGPLv3** (OSI), **RSALv2**, or **SSPLv1** — licensee picks one. Earlier versions (≤7.2) were BSD-3-Clause | Amber | Repo's licensing history is volatile: BSD → SSPL/RSAL (Mar 2024) → tri-license adding AGPLv3 (May 2025). Confirm the version-to-license mapping at procurement |
| Valkey | **BSD-3-Clause** (Linux Foundation fork of Redis 7.2 + ongoing development) | Green | Drop-in Redis replacement created in response to the 2024 license change; backed by AWS, Google Cloud, Oracle |
| ElastiCache Redis / Azure Cache for Redis / Azure Managed Redis | AWS / Azure service terms over the Redis codebase (with provider-side licensing arrangements with Redis Ltd. and/or use of Valkey) | Red managed | AWS ElastiCache moved to Valkey-backed in 2024 |
| DragonflyDB | **BSL 1.1** (converts to Apache 2.0 after 4 years per release) | Amber | Redis-compatible OSS alternative |
| Apache Kafka | Apache 2.0 | Green | — |
| Redpanda | **BSL 1.1** | Amber | Kafka-compatible; community edition has additional restrictions |
| S3 / Azure Blob Storage | AWS / Azure service terms | Red managed | — |
| MinIO Community Edition | AGPLv3 — **repository archived February 2026; community edition no longer maintained** | Red (legacy) | Existing deployments continue to function; new deployments should reconsider. MinIO AIStor (commercial enterprise edition) remains available |
| MinIO alternatives for platform-independent strategy | Ceph (LGPL-2.1 with GPL-2.0+ kernel modules); Garage (AGPLv3); SeaweedFS (Apache 2.0); Cloudflare R2 (commercial) | Green/Amber | SeaweedFS is the cleanest Apache 2.0 substitute |
| DynamoDB / Cosmos DB | AWS / Azure service terms | Red managed | — |
| etcd | Apache 2.0 | Green | Platform-independent metadata catalogue |

**Architectural impact:** The MinIO archival in Feb 2026 is the most material change for this RA. The §19.3 platform-independent column lists MinIO as the OSS object-store equivalent of S3; that recommendation should be updated to SeaweedFS or Ceph for new deployments, with MinIO retained only where existing operations are stable.

### 21.5 Knowledge architecture

| Component | License / terms | Risk | Notes |
|---|---|---|---|
| pgvector | PostgreSQL License | Green | — |
| HNSW algorithm | Algorithm (not patented); reference implementations Apache 2.0 / MIT | Green | — |
| OpenSearch hybrid retrieval (k-NN plugin) | Apache 2.0 | Green | — |
| Cohere Rerank (Bedrock-hosted, D15) | AWS/Cohere service terms via Bedrock | Red managed | Per-call pricing; data-handling per AWS+Cohere agreements |
| Cohere Rerank (Azure via Marketplace) | Azure Marketplace + Cohere terms | Red managed | — |
| `bge-reranker-large` (self-hosted) | **MIT** (BAAI) | Green | Platform-independent option from §19.3 |
| `bge-large-en-v1.5` embeddings | MIT (BAAI) | Green | — |
| `nomic-embed-text` embeddings | Apache 2.0 (Nomic) | Green | — |
| Bedrock Titan embeddings | AWS service terms | Red managed | — |
| Azure OpenAI text-embedding-3-* | Azure service terms | Red managed | — |
| Neo4j (KG, deferred per D14) | Community Edition — **GPLv3**; Enterprise Edition — commercial subscription | Amber/Red | GPLv3 community is acceptable for internal use; clustering and most operational features are Enterprise-only |
| Amazon Neptune | AWS service terms | Red managed | — |
| Memgraph | Community — **BSL 1.1**; Enterprise — commercial | Amber/Red | Alternative to Neo4j for KG |
| RAGAS | Apache 2.0 | Green | — |
| Marquez (OpenLineage) | Apache 2.0 (LF AI & Data project) | Green | D18-aligned |
| OpenLineage spec | Apache 2.0 | Green | — |

### 21.6 Foundation models

Foundation-model licensing is **fundamentally different** from infrastructure licensing. The license attaches to *weights*, *usage*, and often *outputs*, not just code. Read each model's license before commercial use.

| Model | Distribution / license | Risk | Critical clauses |
|---|---|---|---|
| Claude (Anthropic) on Bedrock | AWS+Anthropic service terms via Bedrock | Red managed | Anthropic Acceptable Use Policy applies. Data not used for model training when accessed via Bedrock. |
| Claude on Azure via Marketplace | Azure Marketplace + Anthropic terms | Red managed | Same AUP; Azure billing layer |
| Amazon Nova | AWS service terms | Red managed | — |
| Azure OpenAI (GPT-4-class, GPT-4o-mini) | Azure OpenAI service terms; Microsoft Responsible AI requirements; Azure Content Safety mandatory for some scenarios | Red managed | Stronger residency and compliance controls than direct OpenAI API. Customer Copyright Commitment applies for eligible enterprise scenarios |
| OpenAI direct API | OpenAI ToS | Red managed | Data-handling defaults differ from Azure OpenAI; check enterprise terms |
| Llama 4 (Meta) | **Llama 4 Community License** — commercial use permitted **only** if licensee and affiliates have **<700M MAU**; "Built with Llama" attribution required on derivatives; Acceptable Use Policy applies; **EU-domiciled licensees excluded from multimodal/vision capabilities** under current terms | Amber | Not OSI-approved. The 700M MAU threshold applies to the *entire product or service*, not just AI-feature usage, and aggregates affiliates |
| Llama 3 (Meta) | Llama 3 Community License (same 700M MAU clause, same AUP, no EU multimodal exclusion in the 3.x line) | Amber | — |
| Mistral models (Apache-licensed releases) | **Apache 2.0** for the open-weight models; commercial Mistral Large models have commercial terms | Green/Red | Check the specific model card; Mistral has both Apache-2.0 releases and commercial-only releases |
| Gemma 4 (Google) | **Apache 2.0** with Google's Gemma Prohibited Use Policy | Green | The first major Google open-weight family without a custom license |
| Qwen 3.5 / 3.6 (Alibaba) | Apache 2.0 | Green | — |
| DeepSeek V4 | MIT | Green | — |
| Phi-4 (Microsoft) | MIT | Green | — |

**Architectural impact:** For the platform-independent strategy in §19.3, **Apache 2.0 / MIT-licensed open-weight models (Mistral, Gemma 4, Qwen, DeepSeek, Phi-4) are the clean choice** if compliance gates are strict. Llama 4 is fine for most deployments but introduces legal surface area not present with the Apache/MIT options — enterprise legal teams sometimes flag Llama licenses for review.

### 21.7 Integration plane

| Component | License / terms | Risk | Notes |
|---|---|---|---|
| MCP (Model Context Protocol) | **MIT** (Anthropic; under Linux Foundation since 2025) | Green | Open spec; reference implementations MIT |
| A2A (Agent-to-Agent) | Apache 2.0 (Linux Foundation, 2025) | Green | Open spec |
| AgentCore Gateway | AWS service terms | Red managed | — |
| Azure APIM AI Gateway | Azure service terms; tier-gated (some AI Gateway features Premium-tier only) | Red managed + Amber tier | — |
| Kong AI Gateway (OSS) | Apache 2.0; Kong Konnect (managed control plane) — commercial | Green/Red | Per §19.3 platform-independent option |
| LiteLLM | MIT | Green | — |
| Bifrost | Apache 2.0 (open-source AI gateway) | Green | — |
| Envoy | Apache 2.0 | Green | — |
| API Gateway (AWS) / Azure API Management | Cloud service terms | Red managed | — |

### 21.8 Identity, security, secrets

| Component | License / terms | Risk | Notes |
|---|---|---|---|
| AWS IAM, IAM Identity Center, AgentCore Identity, KMS, Secrets Manager | AWS service terms | Red managed | — |
| Microsoft Entra ID, Entra Agent ID, Azure Key Vault, Defender for Cloud, Sentinel | Azure service terms; **Entra Agent ID requires Entra ID P1/P2 tier** for full governance features | Red managed + Amber tier | Confirm tier at procurement; sponsor/blueprint features carry higher-tier requirements |
| Keycloak | Apache 2.0 | Green | Platform-independent IdP |
| Authentik | MIT | Green | — |
| Dex | Apache 2.0 | Green | — |
| SPIFFE / SPIRE | Apache 2.0 (CNCF) | Green | — |
| HashiCorp Vault | **BUSL 1.1** since Aug 2023; auto-converts to MPL 2.0 four years after each release | Amber | IBM-owned since Feb 2025. The BUSL restriction is on offering Vault as a competing managed service; internal enterprise use is permitted |
| HashiCorp Terraform | **BUSL 1.1** since Aug 2023 | Amber | Same conditions as Vault |
| OpenTofu | MPL 2.0 (Linux Foundation fork of Terraform 1.5.7) | Green | Drop-in Terraform replacement; recommended for platform-independent strategy where BUSL is a blocker |
| OpenBao | MPL 2.0 (Linux Foundation fork of Vault) | Green | Drop-in Vault replacement |
| Cosign / Sigstore | Apache 2.0 (Linux Foundation) | Green | — |
| Trivy | Apache 2.0 (Aqua Security) | Green | — |
| Falco | Apache 2.0 (CNCF) | Green | — |
| Wazuh | GPLv2 | Amber | Strong copyleft; relevant for derivatives, not for use |
| OPA (Open Policy Agent) | Apache 2.0 (CNCF) | Green | D9-locked component |
| OWASP ASI / Top 10 / NIST AI RMF / MITRE ATLAS | CC BY-SA 4.0 (OWASP), public-sector (NIST), public (MITRE) — **standards / catalogues, not software** | Green | Free to reference and align with |

**Architectural impact:** For the platform-independent strategy, BUSL-licensed HashiCorp products are usable but introduce friction. **OpenTofu and OpenBao are the recommended substitutes** unless the engagement is already committed to HashiCorp enterprise licensing post-IBM acquisition.

### 21.9 Observability

| Component | License / terms | Risk | Notes |
|---|---|---|---|
| OpenTelemetry SDKs and Collector | Apache 2.0 (CNCF) | Green | D6-locked standard |
| OpenTelemetry GenAI Semantic Conventions | Apache 2.0 (CNCF) | Green | — |
| AgentCore Observability | AWS service terms | Red managed | — |
| CloudWatch / X-Ray / Application Signals | AWS service terms | Red managed | — |
| Azure Monitor / Application Insights / Foundry Observability | Azure service terms | Red managed | — |
| Prometheus | Apache 2.0 (CNCF) | Green | — |
| Thanos | Apache 2.0 (CNCF) | Green | — |
| Grafana (visualisation) | **AGPLv3** since Apr 2021 (was Apache 2.0) | Amber | Self-hosted use is fine; SaaS providers offering Grafana need a commercial agreement |
| Grafana Loki | AGPLv3 | Amber | Same posture as Grafana |
| Grafana Tempo | AGPLv3 | Amber | Same posture |
| Grafana Mimir | AGPLv3 | Amber | Same posture |
| Loki/Tempo alternatives | VictoriaLogs (Apache 2.0), VictoriaMetrics (Apache 2.0), Jaeger (Apache 2.0) | Green | Apache-2.0 stack avoids AGPL contagion concerns for organisations that flag AGPL |
| Langfuse OSS | **MIT** (core); some Enterprise features (SSO, audit log, RBAC granularity) require commercial license even when self-hosted | Green/Amber | D16-locked. Self-host the OSS core; check feature-by-feature what requires commercial tier |
| Langfuse Cloud | Commercial service terms | Red managed | — |
| Bedrock Evaluations | AWS service terms | Red managed | — |
| Galileo Luna | Commercial (Galileo terms); OSS edition referenced in §19.3 | Red managed / mixed | — |
| Glue Data Quality | AWS service terms | Red managed | — |
| Microsoft Purview | Azure service terms; tier-gated | Red managed + Amber tier | — |
| Great Expectations | Apache 2.0 | Green | Platform-independent alternative |
| RAGAS | Apache 2.0 | Green | — |

### 21.10 Source control, CI/CD, IaC

| Component | License / terms | Risk | Notes |
|---|---|---|---|
| GitHub Enterprise Cloud / Server | Commercial subscription; per-user pricing; tier-gated (Advanced Security, fine-grained RBAC require higher tier) | Red managed + Amber tier | D12-locked |
| GitHub Actions on AWS runners | GitHub commercial + AWS service terms for runners | Red managed | D12 |
| GitLab | Community Edition — **MIT**; Enterprise Edition — commercial subscription with tier-gated features | Green/Red | Platform-independent alternative |
| Gitea | MIT | Green | — |
| Forgejo | MIT (Codeberg fork of Gitea) | Green | — |
| Argo CD | Apache 2.0 (CNCF) | Green | — |
| Argo Workflows | Apache 2.0 (CNCF) | Green | — |
| Tekton | Apache 2.0 (CDF) | Green | — |
| Terraform / OpenTofu | See §21.8 | — | — |
| AWS CDK | Apache 2.0 | Green | — |
| Azure Bicep | MIT | Green | — |
| Pulumi (OSS) | Apache 2.0; Pulumi Cloud is commercial | Green/Red | — |
| Kiro | AWS preview product (service terms) | Red managed | D23-locked |

### 21.11 Standards and protocols (no software license, but worth tabulating)

| Standard | Steward | Use license | Notes |
|---|---|---|---|
| OpenTelemetry GenAI Semantic Conventions | CNCF | Apache 2.0 | D6 |
| OpenLineage | LF AI & Data | Apache 2.0 | D18 |
| FOCUS (FinOps Open Cost and Usage) | FinOps Foundation (Linux Foundation) | Apache 2.0 | §14.2 |
| FinOps Framework 2025 | FinOps Foundation | Creative Commons | §14.2 |
| MCP (Model Context Protocol) | Linux Foundation | MIT (reference impls) | §8.1 |
| A2A (Agent-to-Agent) | Linux Foundation | Apache 2.0 | §6.6, D7 |
| OWASP Top 10 for Agentic Applications | OWASP Foundation | CC BY-SA 4.0 | §11.2 |
| NIST AI RMF | NIST (US Government) | Public sector (free use) | §11.1 |
| MITRE ATLAS | MITRE | Public (free use) | §11.1 |
| AT-FAA / SHIELD | Academic/industry consortium | Public research output | §11.1 |
| W3C Trace Context | W3C | W3C document license (free use) | P10, §9.5 |
| OAuth 2.1 | IETF | RFC (free use) | §8.3 |

### 21.12 License risk by strategy

A roll-up view to support the §19 strategy choice.

| Strategy | Predominant license profile | Compliance friction |
|---|---|---|
| **AWS-native** | Cloud service terms for managed components (Red); permissive OSS for non-coupled components (Green) | Low. Single master service agreement covers most of the surface. AgentCore + Bedrock + EKS terms are well-trodden. |
| **Azure-native** | Cloud service terms for managed components (Red) + tier-gated features (Amber); permissive OSS for non-coupled (Green) | Low–Medium. Watch tier requirements (Entra ID P1/P2 for Agent ID governance; APIM Premium for AI Gateway features). Single EA usually covers most components |
| **Platform-independent** | Heavy Green where Apache 2.0 / MIT options exist; **material Amber** wherever BUSL, AGPLv3, or SSPL-licensed components are pulled in | Medium–High. Requires a license-by-license review and a documented strategy for each Amber component: accept, substitute (OpenTofu for Terraform, Valkey for Redis, SeaweedFS for MinIO), or pay for commercial license. AGPL contagion is the single biggest discussion topic for organisations that ship anything as a service |

**Recommended platform-independent substitutions** for organisations where AGPL or BUSL is a blocker:

| Original | License | Substitute | License |
|---|---|---|---|
| HashiCorp Terraform | BUSL 1.1 | OpenTofu | MPL 2.0 |
| HashiCorp Vault | BUSL 1.1 | OpenBao | MPL 2.0 |
| Redis 8 | AGPLv3 / RSAL / SSPL | Valkey | BSD-3-Clause |
| MinIO (archived) | AGPLv3 (legacy) | SeaweedFS | Apache 2.0 |
| Grafana + Loki + Tempo | AGPLv3 | VictoriaMetrics + VictoriaLogs + Jaeger | Apache 2.0 |
| Elasticsearch | AGPLv3 / SSPL / ELv2 | OpenSearch | Apache 2.0 |

### 21.13 Procurement checklist per deployment

Before signing off on the component list for a specific deployment:

1. **Cloud master service agreement(s) in place** for the chosen strategy (AWS, Azure, or both for hybrid per §19.4)
2. **Per-component license verification** — for every Amber-risk component, capture the current license text, the licensee's named exemption position (e.g., are we above 700M MAU? are we offering this as a competing managed service?), and counsel sign-off
3. **AGPL contagion review** — if any AGPL-licensed component is in the deployment, define which derivative-distribution scenarios apply and which do not. Most enterprise internal use is unaffected; SaaS-style external distribution requires careful scoping
4. **Foundation-model AUP alignment** — for each model used, confirm the Acceptable Use Policy aligns with the use cases planned. Llama models, Anthropic Claude, and Azure OpenAI each have AUPs with different scope
5. **Tier-requirement verification** — GitHub Enterprise tier, Entra ID tier, APIM tier, Langfuse Enterprise features. Several "free" or "self-hosted OSS" components have features behind commercial tiers
6. **Re-license / archive monitoring** — subscribe to the licensor's announcements for every Amber-risk component. The 2023–2026 history shows licenses change with material business impact (HashiCorp, Redis, Elastic, MinIO)
7. **Substitution plan** — for every Amber/Red component, name the substitute and the rough switching cost in case the license becomes a blocker mid-engagement (see §19.6 strategy migration when added)

Add the outcome of this checklist to the per-deployment trade-offs documented under §18 and the strategy commitment under §19.5.

---

## Appendix A — Glossary

### Architecture-session terms

| Term | Definition |
|---|---|
| **A2A** | Agent-to-Agent protocol (Linux Foundation, 2025) — used for agent-to-agent calls across runtimes or organisational boundaries (§6.6, §8.1) |
| **ADLC** | Agent Development Lifecycle — six-phase governance lifecycle every agent passes through (§12.2) |
| **ADR** | Architecture Decision Record |
| **AgentCore** | AWS Bedrock managed substrate: Runtime, Gateway, Identity, Memory, Observability (§2.2). *Azure equivalent: Foundry Agent Service. Platform-independent equivalent: LangGraph + Kubernetes + custom session/identity layer.* |
| **AgentOps** | Operational discipline enforcing the ADLC at runtime; distinct from MLOps and LLMOps (§13) |
| **AKS** | Azure Kubernetes Service |
| **AMS** | Application Management Services (service line) |
| **AOP** | Agentic Operations Platform — IMS's agentic execution platform (§15.4) |
| **APIM** | Azure API Management — Azure's API gateway product; its **AI Gateway** capability covers MCP and LLM gateway functions (§19.3) |
| **APM** | Application Performance Monitoring |
| **App Insights** | Azure Application Insights — Azure's observability surface; the "Agents" view consumes OTel GenAI spans (§19.3) |
| **ACAP** | AD Cloud Agent Platform — AD's AWS-hosted long-running agentic platform (§15.3) |
| **AD** | Application Development (service line) |
| **AD-DWP** | AD Developer Workspace — Kiro IDE on developer laptop (§15.3) |
| **AD-IP** | AD Internal Developer Platform — five-plane IDP hosting AD governance and shared developer tooling (§15.3) |
| **ARP** | Agentic Resolution Platform — multi-step agentic resolution platform pattern (§15.0) |
| **Azure AI Search** | Azure's managed search service; offers native hybrid retrieval (BM25 + vector + semantic ranker) in one product (§19.3) |
| **Bicep** | Azure's IaC language; equivalent of CloudFormation / Terraform |
| **CI** | Configuration Item (from CMDB) |
| **CMDB** | Configuration Management Database |
| **CMP** | Compliance & Monitoring Platform — AMS's compliance and monitoring platform (§15.2) |
| **CPP** | Control Plane Platform — IMS's provisioning/patch/config platform (§15.4) |
| **CWE** | Common Weakness Enumeration (security taxonomy) |
| **Defender for Cloud** | Microsoft's cloud-native vulnerability and posture management; Azure equivalent of AWS Inspector + Security Hub |
| **DiskANN** | High-recall ANN vector index algorithm; Azure-specific extension to pgvector on Azure PostgreSQL Flexible Server (§19.3) |
| **DLP** | Data Loss Prevention |
| **DR** | Disaster Recovery |
| **Entra Agent ID** | Microsoft Entra's purpose-built identity construct for AI agents (GA 2026); Azure equivalent of AgentCore Identity, with added sponsor / blueprint / agent-user concepts (§19.3) |
| **Entra ID** | Microsoft Entra ID — Azure's identity and access management platform (formerly Azure Active Directory) |
| **Event Hubs** | Azure's managed streaming service; Kafka-protocol-compatible; equivalent to MSK |
| **FAG** | FinOps & Asset Governance — IMS's FinOps and asset platform (§15.4) |
| **FOCUS** | FinOps Open Cost and Usage Specification — normalised billing schema (§14.2) |
| **Foundry** | Microsoft Foundry (formerly Azure AI Foundry / Azure AI Studio) — unified Azure platform for AI agents, models, and tools |
| **Foundry Agent Service** | Azure's managed agent runtime; equivalent to AgentCore Runtime; supports BYO frameworks including LangGraph (§19.3) |
| **HITL** | Human-in-the-Loop (§6.5) |
| **HNSW** | Hierarchical Navigable Small World — vector index algorithm used in pgvector (§4.2) |
| **HPA** | Horizontal Pod Autoscaler (Kubernetes) |
| **IDP** | Internal Developer Platform |
| **IMS** | Infrastructure Management Services (service line) |
| **IP** | Intelligence Platform — reactive enrichment, correlation, governance platform pattern (§15.0). *Disambiguation: in AD context "AD-IP" is the Internal Developer Platform, not Intelligence Platform.* |
| **ITSM** | IT Service Management platform (ServiceNow, BMC, Jira Service Management, etc.) |
| **KL** | Knowledge Layer — shared accumulated memory across platforms (§4) |
| **KG** | Knowledge Graph — deferred to Phase 3 per D14 |
| **LiteLLM** | OSS proxy that exposes a unified OpenAI-compatible API across many model providers; common building block in platform-independent strategies (§19.3) |
| **LGTM** | Loki + Grafana + Tempo + Mimir — Grafana Labs OSS observability stack; platform-independent alternative to CloudWatch / Application Insights (§19.3) |
| **MCP** | Model Context Protocol — adapter layer between agent and execution targets (§8.1, §8.2) |
| **MTBF** | Mean Time Between Failures |
| **MTTR / MTTD** | Mean Time To Resolve / Mean Time To Detect |
| **OPA** | Open Policy Agent — declarative policy engine used in the Governance Sidecar (§12.3) |
| **OTel** | OpenTelemetry — telemetry standard; GenAI Semantic Conventions are mandated per D6 |
| **PIR** | Post-Incident Review |
| **PIS** | Predictive Intelligence System — forward-looking failure-prediction platform pattern (§15.0) |
| **Purview** | Microsoft Purview — Azure's data governance and lineage platform; consumes OpenLineage (§19.3) |
| **RAG** | Retrieval-Augmented Generation (§4.3) |
| **RDS** | Amazon Relational Database Service |
| **RRF** | Reciprocal Rank Fusion — score-combination algorithm common in hybrid retrieval |
| **RTO / RPO** | Recovery Time Objective / Recovery Point Objective (§10.5) |
| **SBOM** | Software Bill of Materials; *prompt SBOM* = what the prompt contains and which tools it grants (§13.4) |
| **SDD** | Spec-Driven Development; *constitutional SDD* = SDD layered with mandatory rule constraints (D21) |
| **Sentinel** | Microsoft Sentinel — Azure's cloud-native SIEM; equivalent of AWS Security Hub + GuardDuty (§19.3) |
| **SLI / SLO** | Service Level Indicator / Service Level Objective (§9.6) |
| **SPIFFE / SPIRE** | OSS workload-identity standard and runtime; platform-independent equivalent of AgentCore Identity for service-to-service auth (§19.3) |
| **SPP** | Security Posture Platform — IMS's security platform (§15.4) |
| **vLLM / TGI** | OSS high-throughput LLM serving runtimes; used in platform-independent strategies for self-hosted models (§19.3) |

### Engagement vocabulary (inherited from upstream ticket-analysis phase)

The following terms originate in the upstream ticket-analysis prompt and remain in use during architecture work. They are listed here so cross-references between the analysis output and this document are unambiguous.

| Term | Definition |
|---|---|
| **F-##** | Stable finding identifier assigned by the upstream ticket-analysis phase. Carried unchanged into the architecture session. |
| **UC-##** | Use case identifier from the use-case workbook. |
| **Segment** | Customer-specific *organisational* division (value stream, business unit, service line, line of business, region). Used for finding concentration and per-segment baselines. **Distinct from Domain** — see Appendix B. |
| **Reduction mechanism** | One of eight architecture-neutral labels describing how a finding removes work from operator queues (Filter, Dedupe/aggregate, Correlate-to-parent, Auto-resolve, Recategorise, Routing/enrichment, Indirect, Platform remediation). |
| **Required capability** | The system behaviour that must exist after a change, stated at the capability level — no implementation specifics. |
| **Validation criterion** | A measurable, data-derivable outcome that confirms a capability is working. Architecture-independent acceptance criterion. |
| **Root cause classification** | One of: Configuration gap / Missing automation / Architectural gap / Process failure / Knowledge gap / Data quality issue / Capacity-planning gap. Assigned in the analysis phase. |
| **Strategic objective** | The customer's primary goal from analysis Phase 0 (count reduction / effort reduction / resilience / speed / capability discovery). Determines tier ordering. |
| **Delivery bundle** | A release-scoped packaging of multiple UCs. Architecture-session concern; not a discovery category. |

---

## Appendix B — Segment vs. Domain (keep distinct)

The two terms collide easily in AIOps work (AMS and IMS). Use them precisely:

- **Segment** = customer's *organisational* division. Examples: "North America value stream", "SAP business unit", "Region EMEA", "Retail line of business". Used for finding concentration, segment-level baselines, and per-segment reduction projections. Elicited from the customer in the upstream analysis phase. The customer's own term is preserved.

- **Domain** = *technical ecosystem* within which actions are taken. Examples: "SAP", "Infrastructure", "Database", "Network", "Access", "Endpoint". Used in the ARP-class platforms (AMS-ARP, IMS-AOP, IMS-CPP, IMS-SPP) for per-action-type-per-domain autonomy progression, governance rule selection (OPA per-domain policies), and resolver/runbook scoping.

**Relationship:**

- A single Segment may contain multiple Domains. (The "North America value stream" segment includes SAP, Database, Network, and Access domains.)
- A single Domain spans multiple Segments. (The "Database" domain serves North America, EMEA, and APAC segments alike.)
- **Autonomy tiers progress per-Domain, not per-Segment.** A Database action is allowed up to L3 across all Segments simultaneously once the Database domain reaches L3 maturity; it does not advance Segment by Segment.
- Governance policies (OPA Rego) are written per-Domain; segment-level overrides are exceptions, not the default.

This separation also explains why §15.7 cross-line flows are protocol-driven (A2A / MCP) rather than segment-driven — the technical ecosystem (Domain) is what determines the integration shape.

---

*This reference architecture is a pattern library, not a prescription. Adapt every element to the client's goals, constraints, and existing landscape. Select a cloud strategy explicitly on Day 1 (§19) and surface every other per-deployment trade-off (§18) before designing.*
