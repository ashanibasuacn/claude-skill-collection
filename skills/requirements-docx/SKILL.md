---
name: requirements-docx
description: >
  Generate a formal, well-structured Software Requirements Specification (SRS) document in DOCX format
  from a requirements markdown file, following IEEE 830 best practices. Use this skill whenever the
  user wants to: convert a requirements MD to a Word document, produce a formal SRS, generate a
  requirements DOCX from notes or a draft, formalise requirements into a professional document, or
  says things like "create a requirements doc", "generate SRS from markdown", "turn my requirements
  into a Word document", "convert requirements to DOCX", "I need a formal requirements document",
  or "make a proper SRS". Invoke even if the user's MD is rough or informal — the skill handles
  structure detection and confirms the document plan before generating.
---

# Requirements → DOCX Generator

Converts a requirements markdown file into a formal IEEE 830 Software Requirements Specification (SRS)
in DOCX format. The skill detects the existing structure in the MD, maps it to the SRS template,
confirms the plan with the user, then runs a bundled Python script to produce the finished document.

**Output includes:** cover page · version history table · auto-updating TOC · numbered SRS sections ·
§6 Data Migration (when present) · Appendices (Glossary, Acronyms auto-populated, Open Issues if present)

> **How content is extracted:** The script uses `get_section_full()` which captures ALL content under
> a heading — including every nested sub-heading and its body — until the next sibling or parent
> heading. This means deeply nested MD structures (requirement tables inside `###`/`####` sub-headings)
> are fully included in the output. Fallback keyword matching also uses full hierarchical extraction.

---

## Step 1 — Locate the source file

If the user hasn't provided a path, ask:

> "What's the path to your requirements markdown file?"

Read the file in full once you have it.

---

## Step 2 — Detect the existing structure

Scan all headings (`#`, `##`, `###`, `####`) and note which SRS section each heading maps to:

| If heading title contains… | Maps to SRS section |
|---|---|
| introduction, intro, background | 1. Introduction |
| purpose, objective, system objective | 1.1 Purpose |
| scope | 1.2 Scope |
| definitions, terms, glossary, acronyms | 1.3 Definitions & Acronyms |
| references, reference documents | 1.4 References |
| system overview, product description, context, product perspective | 2.1 Product Perspective |
| product functions, key features, capabilities, functional capabilities | 2.2 Product Functions |
| users, stakeholders, actors, personas, user classes | 2.3 User Classes |
| operating environment, deployment, infrastructure, platform | 2.4 Operating Environment |
| assumptions, dependencies | 2.5 Assumptions & Dependencies |
| constraints, limitations | 2.6 Constraints |
| functional requirements, features, use cases, user stories, epics | 3. Functional Requirements |
| non-functional, nfr, quality attributes, performance, security, scalability | 4. Non-Functional Requirements |
| interface, integration, api, ui requirements, external systems | 5. External Interface Requirements |
| data migration, migration requirements | 6. Data Migration Requirements |
| glossary, definitions, terms and definitions | Appendix A |
| acronyms, abbreviations | Appendix B |
| open issues, gaps, open questions, tbd items, risks, known issues | Appendix C |

Also scan for:
- **Requirement IDs**: `REQ-NNN`, `FR-NNN`, `NFR-NNN`, `DM-NNN`, `DMG-NNN` — note count per section
- **User stories**: lines with "As a … I want … so that …"
- **Glossary entries**: `**Term**: definition`, `- Term: definition`, or Term/Definition tables

---

## Step 3 — Present the proposed document plan

Show the user a mapping table before generating anything. Format it like this:

```
SRS Section                        Source in your MD              Notes
───────────────────────────────────────────────────────────────────────
Cover page                         [metadata — will confirm]
Version history                    [new — v1.0 entry]
Table of Contents                  [auto-generated]
1.  Introduction
  1.1 Purpose                      ← "## Introduction"
  1.2 Scope                        ← "### Scope"
  1.3 Definitions & Acronyms       ← points to Appendices
  1.4 References                   [TBD placeholder]
  1.5 Document Overview            [auto-generated from structure]
2.  Overall Description
  2.1 Product Perspective          ← "## System Overview"
  2.2 Product Functions            ← "### Functional Capabilities"  (nested content included)
  2.3 User Classes                 ← "### Users"
  2.4 Operating Environment        [TBD placeholder]
  2.5 Assumptions & Dependencies   ← "### Assumptions"
  2.6 Constraints                  [TBD placeholder]
3.  Functional Requirements        ← "## Features"  (12 items + all sub-sections)
4.  Non-Functional Requirements    ← "## NFRs"  (5 items)
5.  External Interface Requirements ← "## Integrations"
6.  Data Migration Requirements    ← "## Data Migration"  (optional — omitted if absent)
Appendix A  Glossary               ← 3 definitions extracted
Appendix B  Acronyms               [auto-populated from built-in lookup]
Appendix C  Open Issues            ← "## Gaps" / "## Open Issues"
───────────────────────────────────────────────────────────────────────
TBD placeholders: 3 sections will carry a greyed-out placeholder.
```

Then ask:

> "Does this structure look right? Say **yes** to proceed, or tell me to move/add/remove any sections."

---

## Step 4 — Collect document metadata

Once structure is confirmed, gather these fields. Infer what you can from the MD (title from first `#` heading, project/client from front matter or filename); ask for anything missing:

| Field | Default |
|---|---|
| Document title | `[inferred] Requirements Specification` |
| Project name | [infer or ask] |
| Client / organisation | [ask] |
| Author | [ask] |
| Version | `1.0` |
| Status | `Draft` |
| Date | today |
| Confidentiality | `Internal` |

Confirm all fields with the user in a single compact block before generating.

---

## Step 5 — Generate the DOCX

Build a JSON document plan and call the bundled script. The script needs Python 3 and `python-docx`
(it will auto-install the library if missing).

**Build the plan JSON — section_mapping keys:**

| Key | What it maps to | Notes |
|---|---|---|
| `purpose` | The system objective / purpose heading | |
| `scope` | A dedicated scope section | Often TBD if not a separate heading |
| `product_perspective` | System overview / workflow description | |
| `product_functions` | Functional capabilities summary | Full nested content is captured |
| `user_classes` | Actors / roles / users section | |
| `operating_environment` | Platform / deployment / infrastructure | Often TBD if not a dedicated heading |
| `assumptions` | Assumptions & dependencies | |
| `constraints` | Constraints / limitations | Often TBD |
| `functional_requirements` | The main functional requirements section | All nested `###`/`####` FR sub-sections included |
| `non_functional_requirements` | Non-functional / NFR section | |
| `external_interfaces` | Integration / external interface section | |
| `data_migration` | Data migration section | Produces optional §6 |
| `functional_gaps` | Functional gaps / open questions | → Appendix C |
| `nfr_gaps` | Non-functional gaps | → Appendix C (appended after functional_gaps) |
| `migration_gaps` | Migration-specific gaps | → Appendix C (appended after nfr_gaps) |
| `open_issues` | General open issues / TBD items | → Appendix C (fallback if no gap keys set) |
| `glossary` | Glossary / definitions section | Set to `null` to auto-extract from user_classes |
| `acronyms` | Dedicated acronym section | Set to `null` to auto-populate from built-in lookup |

Set any key to `null` (or omit it) to skip plan-based lookup and rely on keyword fallback or auto-generation.

**Example plan JSON:**
```json
{
  "title": "...",
  "project": "...",
  "client": "...",
  "author": "...",
  "version": "1.0",
  "status": "Draft",
  "date": "YYYY-MM-DD",
  "confidentiality": "Internal",
  "section_mapping": {
    "purpose":                   "### System Objective",
    "scope":                     null,
    "product_perspective":       "## Workflow Description",
    "product_functions":         "### Functional Capabilities",
    "user_classes":              "## Actors & Roles",
    "operating_environment":     null,
    "assumptions":               "## Assumptions",
    "constraints":               null,
    "functional_requirements":   "## Part 1: Functional Requirements",
    "non_functional_requirements": "## Part 2: Non-Functional Requirements",
    "external_interfaces":       "#### B. Integration of the System with:",
    "data_migration":            "## Part 5: Data Migration",
    "functional_gaps":           "## Part 3: Functional Requirement Gaps",
    "nfr_gaps":                  "## Part 4: Non-Functional Requirement Gaps",
    "glossary":                  "## Actors & Roles",
    "acronyms":                  null
  },
  "tbd_sections": ["1.2 Scope", "2.4 Operating Environment", "2.6 Constraints"]
}
```

**Output path:** same directory as the source file, named `[source-basename]-requirements-v1.0.docx`
unless the user specifies otherwise.

**Invoke the script** (the skill's base directory is provided in your context):
```
python "<skill_base_dir>/scripts/generate_docx.py" \
  --source "<source_md_path>" \
  --output "<output_docx_path>" \
  --plan "<plan_json_path>"
```

Write the plan JSON to a temp file alongside the output (e.g., `output-plan.json`) and delete it
after successful generation.

On Windows use `python` or `py`; on Mac/Linux use `python3`.

---

## Step 6 — Fill TBD placeholders (if any)

If the generated document has TBD placeholders (sections where no source heading was found), offer
to draft content for them. Typical TBD sections and how to fill them:

| Section | How to draft it |
|---|---|
| 1.2 Scope | State what the system does (in scope), what it explicitly does NOT do (out of scope), and the business benefit. Derive from the document's purpose statement, scope notes, and assumption/constraint text. |
| 1.4 References | List companion documents mentioned anywhere in the MD (architecture docs, design documents, standards, trackers). Build a numbered table: #, Document name, Description. |
| 2.4 Operating Environment | Extract from NFRs (cloud platform, auth, observability), integration descriptions (mobile device types), and any deployment notes. Cover: browser/device, cloud platform, auth mechanism, key integrations. |
| 2.6 Constraints | Derive from scope limitations, regulatory NFRs (PII, compliance), architecture mandates, and assumptions. Each constraint should be one bullet: the rule + its source ID. |

To insert drafted content into the DOCX, write a small Python patch script that:
1. Opens the DOCX with `python-docx`
2. Finds each paragraph whose text contains `[TBD — <label>]`
3. Inserts new paragraphs/tables immediately after it using `_p.getparent().insert()`
4. Removes the TBD paragraph
5. Saves the file

Delete the patch script after successful execution.

---

## Step 7 — Report completion

Tell the user:
- Full path to the generated DOCX
- How many sections were populated from source vs TBD placeholders
- Count of requirement IDs, tables, and open issues captured
- "Open the document in Word and press **Ctrl+A → F9** to refresh the Table of Contents."
- If TBD placeholders remain, offer to draft and insert them now (Step 6)

Read `references/srs-structure.md` if you need guidance on what each SRS section should contain.
