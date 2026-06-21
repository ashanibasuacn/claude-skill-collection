# CLAUDE.md

Guidance for Claude Code working in this repository.

## What This Is

A Claude Code **plugin** named **`ppes-rde`** — no application code, entirely prompt-based skill authoring. The plugin bundles every skill; once installed, skills resolve under the `ppes-rde` namespace (e.g. `ppes-rde:aiops-analysis`, `ppes-rde:requirements-docx`). Two skill families:

- **`AIOps/`** — a 3-session AIOps consulting engagement (analysis → architecture → estimation). Sessions chain: each session's outputs feed the next.
- **`sdlc/`** — `requirements-docx`, converts a requirements markdown file into a formal IEEE 830 SRS DOCX.

**Plugin wiring:**
- `.claude-plugin/plugin.json` — manifest; `name` must stay `ppes-rde` (it sets the install namespace).
- `.claude-plugin/marketplace.json` — makes the repo installable: `/plugin marketplace add <repo>` then `/plugin install ppes-rde@ppes-rde`.
- `skills/` — **the single source of truth.** All four skills live here; this is what the plugin installs. Edit skills here.
- `AIOps/` and `sdlc/` — family-level READMEs plus the local `*-workspace/` eval dirs (gitignored). No skill source or `.skill` archives live here anymore.

## Skill Structure

Every skill under `skills/` follows the same layout:

```
skills/
└── <skill-name>/            # Source directory (plugin loads from here)
    ├── SKILL.md             # YAML frontmatter (name, description) + workflow
    ├── references/          # Docs loaded on demand (phase-a.md, phase-b.md, ref-*.md)
    └── scripts/             # Optional bundled executables (e.g. generate_docx.py)
```

SKILL.md detects the current phase/step and reads the relevant reference file in full.

**AIOps two-phase pattern:**
- **Phase A** — Interactive interview → 3–4 numbered markdown files (`s#-##-[client]-[description].md`) in `ai_ticket_analysis/`.
- **Phase B** — Reads Phase A outputs → styled HTML report (Accenture-branded).

## The Three AIOps Sessions

| Session | Skill | Input | Output files |
|---|---|---|---|
| 1 — Analysis | `aiops-analysis` | Ticket data, SRE reports | `s1-01`–`s1-04` |
| 2 — Architecture | `aiops-architecture` | `s1-01`, `s1-03`, `s1-04` | `s2-01`–`s2-04` |
| 3 — TEP (estimation) | `aiops-tep` | Session 2 outputs | `s3-01`–`s3-03` |

## Key Conventions (AIOps)

- **Output naming:** `s#-##-[client]-[description].md` (`#`=session, `##`=sequence, `client`=engagement name).
- **Output directory:** `ai_ticket_analysis/` (confirm with user before first write).
- **UC taxonomy:** 14 use cases UC-01–UC-14, defined in analysis, referenced through architecture/estimation.
- **Findings traceability:** Findings `F-01`, `F-02`, … from Session 1 are tied to use cases in Session 2.
- **T-shirt sizing (Session 3):** Default `XS=1–2pd, S=3–5pd, M=6–10pd, L=11–20pd, XL=21–40pd`; user can override at session start.
- **Reference architecture lock:** `skills/aiops-architecture/references/ref-agentic-architecture.md` is a locked ~40k-line pattern library covering all architectural concerns (Compute, Data, Knowledge, ML Pipelines, Agent Orchestration, Memory, Integration, Observability, Reliability, Security, Governance, Quality/AgentOps, FinOps) across three deployment strategies (AWS-native, Azure-native, platform-independent). Do not edit casually.

## Distribution

No build step. The plugin serves skills straight from `skills/`, so edits there are live — users pull them with `/plugin marketplace update ppes-rde`. Commit and push to publish.

If you ever need a standalone single-skill `.skill` bundle (a ZIP), build it on demand — PowerShell's `Compress-Archive` only writes `.zip`, so compress then rename:

```powershell
Compress-Archive -Path skills/aiops-analysis/* -DestinationPath aiops-analysis.zip -Force
Rename-Item aiops-analysis.zip aiops-analysis.skill -Force
```

## Evaluating Skills

Eval/benchmark artifacts live in `<skill-name>-workspace/` directories (gitignored via `*-workspace/`). Each `iteration-N/eval-*/` folder holds `with_skill/` and `without_skill/` runs with `outputs/`, `grading.json`, `timing.json`; results aggregate into `benchmark.json` / `benchmark.md`.

`grading.json` expectations use fields `text`, `passed`, `evidence`, plus a `summary` block (`pass_rate`, `passed`, `failed`, `total`) — the aggregator and viewer depend on these exact names.

## Working with skill-creator

Use `/skill-creator` to create, iterate on, or eval skills. It understands the SKILL.md schema and the patterns used here.
