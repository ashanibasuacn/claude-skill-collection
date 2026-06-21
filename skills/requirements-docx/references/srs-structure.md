# IEEE 830 SRS Section Reference

This file describes what each section of the Software Requirements Specification should contain.
Read it when you need guidance on filling TBD sections or explaining the structure to the user.

---

## Cover Page
- Document title (usually "[System/Product Name] Software Requirements Specification")
- Project and client/organisation names
- Author(s), version, status (Draft / Review / Approved), date
- Confidentiality classification

## Document Control — Version History
A table tracking every revision:
| Version | Date | Author | Status | Description of Changes |

Start at v1.0 Draft. Increment minor (1.1, 1.2) for revisions within a status; increment major (2.0) on approval.

---

## 1. Introduction

### 1.1 Purpose
What this document is for, and who the intended audience is. One or two paragraphs.
Example: "This SRS defines the functional and non-functional requirements for the Acme Invoice Portal, intended for the development team, QA, and the client's product owner."

### 1.2 Scope
Name the product/system. State what it does (high level), what it does NOT do (boundaries), and the business benefit it delivers. Keep it to half a page.

### 1.3 Definitions, Acronyms, and Abbreviations
Technical terms, domain vocabulary, and abbreviations used in this document.
Best practice: cross-reference Appendix A (Glossary) and Appendix B (Acronyms) here rather than duplicating.

### 1.4 References
Other documents this SRS depends on: architecture documents, standards cited, API specs, earlier requirements. Use a numbered list with document title, version, and date.

### 1.5 Document Overview
One paragraph describing the structure of the rest of the document. Auto-generated if not present in the source.

---

## 2. Overall Description

### 2.1 Product Perspective
Where the product fits in the larger system landscape. Describe relationships to other systems (ERP, CRM, external APIs). Include a context diagram if available.

### 2.2 Product Functions
A high-level summary of the major functions the product performs. This is a summary — details go in Section 3. A bulleted list of 5–10 items is sufficient.

### 2.3 User Classes and Characteristics
Who will use the system? Describe each user class: role, technical level, frequency of use, access privileges. Example user classes: Administrator, End User, Reporting Analyst, External API Consumer.

### 2.4 Operating Environment
Hardware, OS, browser, cloud platform, mobile, network constraints. Be specific: "Chrome 120+, Firefox 115+, responsive down to 375 px width" or "AWS EKS, Kubernetes 1.28".

### 2.5 Assumptions and Dependencies
Assumptions the team is making that could affect the requirements (e.g., "Single Sign-On provider will be available at project start"). External dependencies the system relies on (e.g., "Availability of the client's HR data feed via REST API").

### 2.6 Constraints
Design, regulatory, or business constraints the solution must stay within: budget, timeline, technology mandates, compliance (GDPR, HIPAA, PCI-DSS), accessibility standards (WCAG 2.1 AA).

---

## 3. Functional Requirements
The core of the document. Organise by feature, module, epic, or use case — whichever maps most clearly to the user's MD structure. Each requirement should:
- Have a unique ID (REQ-001 / FR-001)
- Use a consistent verb: "The system **shall**…" (mandatory) or "The system **should**…" (desirable)
- Be unambiguous, verifiable, and atomic (one behaviour per requirement)

Common sub-structure:
- **3.1 [Feature/Module Name]** — FR-001, FR-002, …
- **3.2 [Feature/Module Name]** — FR-010, FR-011, …

---

## 4. Non-Functional Requirements
Quality attributes the system must exhibit. Organised by category:

### 4.1 Performance
Response times, throughput, concurrency. Be quantitative: "The dashboard must load within 2 seconds at the 95th percentile under 500 concurrent users."

### 4.2 Security
Authentication, authorisation, encryption, audit logging, penetration testing requirements, compliance mandates.

### 4.3 Usability
Accessibility standards, maximum error rate targets, onboarding time, mobile/responsive requirements.

### 4.4 Reliability and Availability
Uptime SLA, RTO/RPO, disaster recovery requirements. Example: "99.9% uptime (≤8.7 h downtime/year)."

### 4.5 Scalability
Growth capacity: data volume, user count, transaction rate over 1–3 years.

### 4.6 Maintainability (optional)
Code coverage floor, deployment frequency, tech-stack constraints if relevant.

---

## 5. External Interface Requirements

### 5.1 User Interfaces
Key screens/flows (wireframes by reference if available). Layout constraints, responsive breakpoints, branding guidelines.

### 5.2 Hardware Interfaces
Physical devices the system interfaces with (printers, scanners, IoT sensors, card readers).

### 5.3 Software Interfaces
Third-party APIs, internal services, file formats exchanged. For each: name, version, protocol, data format, authentication method.

### 5.4 Communication Interfaces
Network protocols, message queues, email/SMS gateways, data encryption in transit.

---

## Appendix A: Glossary
Table of domain terms and their definitions.
Format: two-column table — Term | Definition.
List in alphabetical order. Bold the term column.

## Appendix B: Acronyms and Abbreviations
Table: Acronym | Expansion.
List in alphabetical order.

## Appendix C: Open Issues (optional)
Requirements that are still under discussion, deferred decisions, or items needing stakeholder input.
Format: ID | Issue Description | Owner | Target Resolution Date.
Remove this appendix from the final approved version once all items are resolved.
