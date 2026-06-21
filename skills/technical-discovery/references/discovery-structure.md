# Current Landscape / Technical Discovery — Section Content Guide

What each section of the discovery document should contain. Use this to judge whether a source
heading really fits a section, and to help the user draft content for TBD placeholders. This mirrors
how current-state / baseline-architecture assessments are structured in practice (e.g. TOGAF
baseline architecture, application portfolio assessments).

## 1. Executive Summary
A non-technical, one-page synopsis: what was assessed, the headline state of the landscape, and the
2–4 most important observations. Write this last, after the body is settled. It should make sense to
a stakeholder who reads nothing else.

## 2. Discovery Scope & Approach
- **2.1 Scope** — what is in and out of scope (systems, domains, time period, environments).
- **2.2 Approach & Sources** — how the discovery was conducted (workshops, doc review, code/repo
  inspection, interviews) and the inputs relied on. The Source Inventory appendix lists the files.
- **2.3 Assumptions** — anything taken as given because it couldn't be verified during discovery.

## 3. Business Context
- **3.1 Business Capabilities & Domain Overview** — the business capabilities the systems support
  and the domain language. Keeps the technical findings anchored to business value.
- **3.2 Business & Process Flows** — the key end-to-end flows (e.g. order-to-cash, onboarding),
  ideally as numbered steps or a flow description. Diagrams referenced in source go here.
- **3.3 Users & Stakeholders** — user classes / personas, their goals, volumes, and the
  stakeholders who own or depend on the systems.

## 4. Application & Architecture Landscape
The technical heart of the document.
- **4.1 System & Application Inventory** — a table of applications/systems: name, purpose, owner,
  lifecycle status (strategic / sustain / retire), and criticality. The backbone of the assessment.
- **4.2 Architecture Overview** — the as-is logical/component architecture: major components, how
  they fit together, deployment topology, and notable patterns or anti-patterns.
- **4.3 Integration Landscape** — interfaces between systems and to external parties: integration
  style (API, file, event/queue), protocols, sync/async, and known brittle points.

## 5. Technology Stack
Languages, frameworks, runtimes, and their versions — flag end-of-life / unsupported versions, which
are a common discovery finding. A matrix of component → technology → version works well.

## 6. Tools & Platforms
- **6.1 Development & Delivery** — source control, build, CI/CD, release process, test tooling.
- **6.2 Hosting & Infrastructure** — where things run: cloud/on-prem, regions, compute model
  (VMs / containers / serverless), networking essentials.
- **6.3 Monitoring & Observability** — logging, metrics, tracing, alerting, dashboards, and gaps in
  visibility (often a key pain point).

## 7. Data Landscape
- **7.1 Data Model** — the core entities and their relationships; the conceptual/logical model.
  Reference ERDs from source here.
- **7.2 Data Stores & Flows** — databases and stores (type, purpose, ownership), how data moves
  between them, data volumes, and any master-data / data-quality concerns.

## 8. Operational & Non-Functional Profile
- **8.1 Security Posture** — authn/authz approach, identity, secrets management, compliance regimes,
  and known security gaps.
- **8.2 Availability & Resilience** — SLAs/SLOs, redundancy, failover, backup/DR posture.
- **8.3 Performance & Scalability** — current performance characteristics, known bottlenecks, scaling
  approach and limits.

## 9. Known Issues, Pain Points & Technical Debt
The candid list: recurring incidents, fragile areas, manual workarounds, deprecated tech, and
accumulated technical debt. Where possible note impact and frequency. This section often drives the
recommendations.

## 10. Constraints, Risks & Dependencies
Constraints the landscape imposes (licensing, regulatory, vendor lock-in, skills), risks to
stability or change, and external dependencies (third parties, upstream/downstream systems).

## 11. Observations & Recommendations
Synthesis: the most significant observations and pragmatic, prioritised recommendations or next
steps. Tie each recommendation back to a finding from sections 4–10 so it's defensible. Keep
recommendations directional unless the engagement scope includes detailed remediation planning.

## Appendices
- **A. Glossary** — domain and technical terms used in the document.
- **B. Acronyms and Abbreviations** — expanded forms.
- **C. Source Inventory** — the markdown files the document was assembled from (auto-generated).
- **D. Open Questions** — unresolved questions to chase down as discovery continues.

## Drafting TBD sections
When helping the user fill a TBD placeholder, pull from the other sections first (e.g. infer 6.2
Hosting from an Architecture Overview that names the cloud), then ask targeted questions rather than
inventing specifics. Never fabricate concrete facts (versions, vendors, SLAs) — if unknown, leave the
placeholder or record the gap in Appendix D.
