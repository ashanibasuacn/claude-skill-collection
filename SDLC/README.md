# SDLC Skills

A set of [Claude Code](https://claude.com/claude-code) skills for software development lifecycle tasks — turning raw artefacts (requirements notes, architecture decisions, tickets) into formal, professional documents.

## Skills

| Skill | File | What it does |
|---|---|---|
| `requirements-docx` | [`requirements-docx.skill`](requirements-docx.skill) | Converts a requirements markdown file into a formal IEEE 830 SRS document (DOCX) with cover page, version history, TOC, numbered sections, and appendices. |

---

## requirements-docx

Converts a requirements markdown file — however rough or polished — into a formal **Software Requirements Specification (DOCX)** following the IEEE 830 standard.

### What you get

| Section | Contents |
|---|---|
| Cover page | Title, project, client, author, version, status, date, confidentiality |
| Document Control | Version history table (v1.0 entry pre-populated) |
| Table of Contents | Auto-updating Word TOC field (Ctrl+A → F9 to refresh) |
| 1. Introduction | Purpose · Scope · Definitions & Acronyms · References · Document Overview |
| 2. Overall Description | Product Perspective · Product Functions · User Classes · Operating Environment · Assumptions & Dependencies · Constraints |
| 3. Functional Requirements | Content from your MD, or TBD placeholder |
| 4. Non-Functional Requirements | Content from your MD, or structured TBD subsections (Performance · Security · Usability · Reliability · Scalability) |
| 5. External Interface Requirements | User · Hardware · Software · Communication Interfaces |
| Appendix A | Glossary (terms auto-extracted from MD definitions) |
| Appendix B | Acronyms and Abbreviations |
| Appendix C | Open Issues (included only if detected in source) |

Sections not found in your source MD are inserted as greyed-out TBD placeholders so the document is complete and ready for review.

### How to use it

Invoke by intent — say anything like:

- *"Turn my requirements MD into a Word document"*
- *"Generate a formal SRS from requirements-draft.md"*
- *"I need a proper requirements document for governance review"*
- *"Make a DOCX from my requirements notes"*

The skill will:

1. Ask for the path to your requirements markdown file (if not already given)
2. Read the file and detect its existing structure
3. Show you a document plan — mapping each detected heading to the correct SRS section, with TBD gaps flagged
4. Ask you to confirm or adjust the plan
5. Collect cover page metadata (title, author, client name, version, date)
6. Run the bundled `generate_docx.py` script (auto-installs `python-docx` if needed)
7. Report the output path and remind you to refresh the TOC in Word

### Requirements

- **Python 3** must be available in the terminal (`python` / `python3` / `py`)
- `python-docx` is auto-installed on first run if not already present
- Works on Windows, Mac, and Linux

### Output naming

Default: `[source-basename]-requirements-v1.0.docx` in the same directory as the source file.

Example: `portal-requirements.md` → `portal-requirements-requirements-v1.0.docx`

You can specify a different path during the confirmation step.

---

## Layout

```
sdlc/
├── requirements-docx/              # Skill source
│   ├── SKILL.md                    # Frontmatter + 6-step workflow
│   ├── scripts/
│   │   └── generate_docx.py        # DOCX generation script (python-docx)
│   └── references/
│       └── srs-structure.md        # IEEE 830 section content guide
├── requirements-docx.skill         # Packaged skill (ZIP)
└── requirements-docx-workspace/    # Evals + benchmark results
    ├── evals/evals.json
    ├── sample-inputs/
    └── iteration-1/
```

## Packaging (after editing source)

```powershell
# From the sdlc/ directory
Compress-Archive -Path requirements-docx\* -DestinationPath requirements-docx.zip -Force
Rename-Item -Path requirements-docx.zip -NewName requirements-docx.skill -Force
```

## Developing these skills

See the repository [CLAUDE.md](../CLAUDE.md) for skill authoring guidance and the evaluation workflow. Use `/skill-creator` to iterate on any skill in this folder.
