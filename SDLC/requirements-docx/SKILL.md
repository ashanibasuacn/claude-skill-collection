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
Appendices (Glossary, Acronyms, Open Issues if present)

---

## Step 1 — Locate the source file

If the user hasn't provided a path, ask:

> "What's the path to your requirements markdown file?"

Read the file in full once you have it.

---

## Step 2 — Detect the existing structure

Scan all headings (`#`, `##`, `###`) and note which SRS section each heading maps to:

| If heading title contains… | Maps to SRS section |
|---|---|
| introduction, intro, background | 1. Introduction |
| purpose | 1.1 Purpose |
| scope | 1.2 Scope |
| definitions, terms, glossary, acronyms | 1.3 Definitions & Acronyms |
| references, reference documents | 1.4 References |
| system overview, product description, context, product perspective | 2.1 Product Perspective |
| product functions, key features, capabilities, system functions | 2.2 Product Functions |
| users, stakeholders, actors, user classes | 2.3 User Classes |
| operating environment, deployment, infrastructure, platform | 2.4 Operating Environment |
| assumptions, dependencies | 2.5 Assumptions & Dependencies |
| constraints, limitations | 2.6 Constraints |
| functional requirements, features, use cases, user stories, epics, requirements | 3. Functional Requirements |
| non-functional, nfr, quality attributes, performance, security, scalability, usability, reliability | 4. Non-Functional Requirements |
| interface, integration, api, ui requirements, external systems | 5. External Interface Requirements |
| glossary, definitions, terms and definitions | Appendix A |
| acronyms, abbreviations | Appendix B |
| open issues, tbd, risks, known issues | Appendix C |

Also scan for:
- **Requirement IDs**: `REQ-NNN`, `FR-NNN`, `NFR-NNN`, `US-NNN` — note count per section
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
  1.3 Definitions & Acronyms       ← 3 definitions detected
  1.4 References                   [TBD placeholder]
  1.5 Document Overview            [auto-generated from structure]
2.  Overall Description
  2.1 Product Perspective          ← "## System Overview"
  2.2 Product Functions            [TBD placeholder]
  2.3 User Classes                 ← "### Users"
  2.4 Operating Environment        [TBD placeholder]
  2.5 Assumptions & Dependencies   ← "### Assumptions"
  2.6 Constraints                  [TBD placeholder]
3.  Functional Requirements        ← "## Features"  (12 items)
4.  Non-Functional Requirements    ← "## NFRs"  (5 items)
5.  External Interface Requirements [TBD placeholder]
Appendix A  Glossary               ← 3 definitions extracted
Appendix B  Acronyms               [empty table — populate later]
───────────────────────────────────────────────────────────────────────
TBD placeholders: 5 sections will carry a greyed-out placeholder.
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

**Build the plan JSON:**
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
    "purpose":                    "### Purpose",
    "scope":                      "### Scope",
    "functional_requirements":    "## Features",
    "non_functional_requirements":"## NFRs",
    "product_perspective":        "## System Overview",
    "user_classes":               "### Users",
    "assumptions":                "### Assumptions",
    "glossary":                   null,
    "acronyms":                   null
  },
  "tbd_sections": ["1.4 References", "2.2 Product Functions", "2.4 Operating Environment"]
}
```

Include only sections where the user's MD has content in `section_mapping`; set to `null` for sections
with no source. List every TBD section in `tbd_sections`.

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

## Step 6 — Report completion

Tell the user:
- Full path to the generated DOCX
- How many sections were populated from source vs TBD placeholders
- "Open the document in Word and press **Ctrl+A → F9** to refresh the Table of Contents."
- If TBD placeholders exist, offer to help draft content for any of them now

Read `references/srs-structure.md` if you need guidance on what each SRS section should contain
or how to help the user fill in TBD sections.
