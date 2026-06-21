---
name: technical-discovery
description: >
  Generate a formal Current Landscape / Technical Discovery document in DOCX format from one or
  more existing markdown files, following industry best practice for current-state assessments.
  Use this skill whenever the user wants to: produce a technical discovery document, a current
  landscape / current-state assessment, an as-is architecture or baseline architecture document,
  a system/application landscape write-up, or consolidate discovery notes into a formal Word
  deliverable. Trigger on phrases like "create a technical discovery doc", "current landscape
  document", "as-is / current-state assessment", "document our existing architecture and tech
  stack", "turn these discovery notes into a Word document", or "landscape document from my md
  files". The skill reads the source file(s), detects what landscape areas they cover
  (architecture, technology, tools, users, business flows, data model, known issues, platform,
  etc.), confirms BOTH the input file list AND the proposed document structure with the user,
  then generates the DOCX. Invoke even if the source notes are rough or spread across several files.
---

# Current Landscape / Technical Discovery → DOCX Generator

Consolidates one or more discovery markdown files into a formal **Current Landscape / Technical
Discovery** document (neutral, client-agnostic styling). The skill confirms which inputs to use,
detects which landscape areas the sources cover, maps them onto a best-practice discovery
structure, confirms the plan with the user, then runs a bundled Python script to produce the DOCX.

**Why confirm twice:** discovery inputs are often scattered (several notes files, partial coverage),
so the user must confirm *which files* are in scope before reading, and *which structure* before
generating — otherwise the document silently misses sources or invents sections. Sections with no
source content become explicit TBD placeholders rather than fabricated prose.

**Output includes:** cover page · version history · auto-updating TOC · numbered discovery sections
· appendices (Glossary, Acronyms, **Source Inventory**, Open Questions).

---

## Step 1 — Locate and confirm the input files

The user may give a single file, several files, or a folder. Resolve it, then **confirm the exact
list before reading**, because which files are in scope changes the whole document.

- If given a folder or glob, list the `.md` files you found and ask the user to confirm/prune:
  > "I found these markdown files — confirm which to include in the discovery document:
  > 1. `infra-notes.md` 2. `app-inventory.md` 3. `data-model.md`. Use all, or a subset?"
- If given one file, confirm it's the only source or whether others should join it.

Once confirmed, **read every included file in full.** Note which file each major heading came from —
you'll cite the source set in the plan and the document's Source Inventory appendix.

---

## Step 2 — Detect what the sources cover

Scan all headings (`#`–`####`) across every confirmed file and map each to a discovery section.
A heading title containing any of the keywords maps to that section:

| If a heading mentions… | Maps to discovery section |
|---|---|
| executive summary, summary, overview, background, introduction | 1. Executive Summary |
| scope, objectives, in scope / out of scope | 2.1 Scope |
| approach, method, methodology, sources, discovery process | 2.2 Approach & Sources |
| assumptions, dependencies | 2.3 Assumptions |
| business capability, business context, domain, business overview | 3.1 Business Capabilities & Domain |
| business flow, process flow, workflow, user journey, processes | 3.2 Business & Process Flows |
| users, user classes, stakeholders, actors, personas, roles | 3.3 Users & Stakeholders |
| application/system inventory, applications, systems, components | 4.1 System & Application Inventory |
| architecture, solution/system architecture, component, logical view | 4.2 Architecture Overview |
| integration, interfaces, API, messaging, external systems | 4.3 Integration Landscape |
| technology stack, tech stack, languages, frameworks, runtime | 5. Technology Stack |
| CI/CD, build, pipeline, devops, development tools | 6.1 Development & Delivery |
| hosting, infrastructure, platform, cloud, deployment, environment | 6.2 Hosting & Infrastructure |
| monitoring, observability, logging, alerting, telemetry | 6.3 Monitoring & Observability |
| data model, entities, schema, ERD, domain model | 7.1 Data Model |
| data store, database, data flow, data sources, storage | 7.2 Data Stores & Flows |
| security, authentication, authorization, compliance, IAM | 8.1 Security Posture |
| availability, resilience, reliability, disaster recovery, SLA | 8.2 Availability & Resilience |
| performance, scalability, capacity, throughput, latency | 8.3 Performance & Scalability |
| known issues, pain points, technical debt, gaps, limitations | 9. Known Issues & Technical Debt |
| constraints, risks, dependencies | 10. Constraints, Risks & Dependencies |
| recommendations, observations, next steps, opportunities, findings | 11. Observations & Recommendations |
| glossary, definitions, terms | Appendix A: Glossary |
| acronyms, abbreviations | Appendix B: Acronyms |
| open questions, open issues, to be confirmed | Appendix D: Open Questions |

Also note any tables (inventories, tech matrices) and diagrams referenced — these usually belong in
sections 4, 5, or 7. It's fine for one source heading to feed a section and for many sections to have
no source at all; that's expected in early discovery.

---

## Step 3 — Present the proposed document plan

Show the mapping before generating anything, so the user can see coverage and gaps. Format it like:

```
Discovery Section                       Source                          Notes
──────────────────────────────────────────────────────────────────────────────────────
Cover · Document Control · TOC          [metadata — will confirm]
1.  Executive Summary                   ← infra-notes.md "## Overview"
2.  Discovery Scope & Approach
  2.1 Scope                             ← app-inventory.md "## Scope"
  2.2 Approach & Sources                [TBD placeholder]
  2.3 Assumptions                       [TBD placeholder]
3.  Business Context
  3.1 Business Capabilities & Domain    [TBD placeholder]
  3.2 Business & Process Flows          ← app-inventory.md "## Order Flow"
  3.3 Users & Stakeholders              ← app-inventory.md "## Users"
4.  Application & Architecture Landscape
  4.1 System & Application Inventory    ← app-inventory.md "## Applications" (table, 9 systems)
  4.2 Architecture Overview            ← infra-notes.md "## Architecture"
  4.3 Integration Landscape            ← infra-notes.md "## Integrations"
5.  Technology Stack                    ← infra-notes.md "## Tech Stack"
6.  Tools & Platforms
  6.1 Development & Delivery            [TBD placeholder]
  6.2 Hosting & Infrastructure          ← infra-notes.md "## Hosting"
  6.3 Monitoring & Observability        [TBD placeholder]
7.  Data Landscape
  7.1 Data Model                        ← data-model.md "## Entities"
  7.2 Data Stores & Flows               ← data-model.md "## Stores"
8.  Operational & Non-Functional Profile [TBD: 8.1 Security, 8.2 Availability, 8.3 Performance]
9.  Known Issues & Technical Debt        ← infra-notes.md "## Pain Points"
10. Constraints, Risks & Dependencies    [TBD placeholder]
11. Observations & Recommendations       [TBD placeholder]
Appendix A Glossary · B Acronyms · C Source Inventory (3 files) · D Open Questions
──────────────────────────────────────────────────────────────────────────────────────
TBD placeholders: N sections will carry a greyed-out placeholder.
```

Then ask:

> "Does this structure look right? Say **yes** to proceed, or tell me to move/add/remove/merge any
> sections, or re-map a source. I can also drop sections you don't want in this document."

Be willing to tailor — not every discovery needs all 11 sections. If the sources are all infra/tech
with no business content, offer to drop section 3, etc.

---

## Step 4 — Collect document metadata

Once the structure is confirmed, gather these. Infer what you can (title/project from the sources or
folder name); ask for anything missing. **Project name drives the output filename.**

| Field | Default |
|---|---|
| Document title | `Current Landscape — Technical Discovery` |
| Project name | [infer or ask — drives filename] |
| Client / organisation | [ask] |
| Author | [ask] |
| Version | `1.0` |
| Status | `Draft` |
| Date | today |
| Confidentiality | `Internal` |

Confirm all fields in one compact block before generating.

---

## Step 5 — Generate the DOCX

Build a plan JSON and call the bundled script. It needs Python 3 and `python-docx` (auto-installed
if missing). The script accepts **multiple** source files after `--source`.

**Plan JSON shape:**
```json
{
  "title": "Current Landscape — Technical Discovery",
  "project": "...",
  "client": "...",
  "author": "...",
  "version": "1.0",
  "status": "Draft",
  "date": "YYYY-MM-DD",
  "confidentiality": "Internal",
  "source_files": ["infra-notes.md", "app-inventory.md", "data-model.md"],
  "section_mapping": {
    "executive_summary":     "## Overview",
    "architecture_overview": "## Architecture",
    "technology_stack":      "## Tech Stack",
    "system_inventory":      "## Applications",
    "data_model":            "## Entities"
  },
  "tbd_sections": ["2.2 Approach & Sources", "3.1 Business Capabilities", "..."]
}
```

The `section_mapping` keys are: `executive_summary, scope, approach, assumptions,
business_capabilities, business_flows, users_stakeholders, system_inventory, architecture_overview,
integration_landscape, technology_stack, dev_delivery, hosting_infra, monitoring, data_model,
data_stores, security, availability, performance, known_issues, constraints_risks, recommendations,
glossary, acronyms, open_questions`. Include only keys where the source has content; map each to the
**exact heading text** (with its `##` prefix) from the source. The script also keyword-matches as a
fallback, so a perfect mapping isn't required — but an explicit mapping is more reliable.

**Output path & name:** default `[project-slug]-technical-discovery-v1.0.docx` in the working
directory (or alongside the sources), unless the user specifies otherwise.

**Invoke the script** (skill base directory is in your context). List every confirmed source file:
```
python "<skill_base_dir>/scripts/generate_docx.py" \
  --source "<file1.md>" "<file2.md>" "<file3.md>" \
  --output "<output_docx_path>" \
  --plan "<plan_json_path>"
```
Write the plan JSON to a temp file (e.g. `output-plan.json`) and delete it after success.
On Windows use `python` or `py`; on Mac/Linux use `python3`.

---

## Step 6 — Report completion

Tell the user:
- Full path to the generated DOCX and the list of source files folded in
- How many sections were populated from source vs left as TBD placeholders
- "Open it in Word and press **Ctrl+A → F9** to refresh the Table of Contents."
- Offer to help draft content for any TBD section now

Read `references/discovery-structure.md` if you need guidance on what each discovery section should
contain or how to help the user fill in TBD sections.
