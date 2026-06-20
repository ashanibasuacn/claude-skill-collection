# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A collection of Claude Code skills for delivering a structured 3-session **AIOps consulting engagement**. The skills guide a client from raw operational ticket/log data through use-case analysis, architecture design, and implementation estimation. All three sessions chain together: each session's outputs are the next session's inputs.

The skills live under `AIOps/` (currently untracked by git). There is no application code — this is entirely prompt-based skill authoring.

## Skill Structure

Every skill follows an identical layout:

```
AIOps/
├── <skill-name>/            # Source directory
│   ├── SKILL.md             # Frontmatter + workflow instructions
│   └── references/
│       ├── phase-a.md       # Interactive session prompt
│       ├── phase-b.md       # HTML report generation prompt
│       └── [ref-*.md]       # Optional locked reference docs
└── <skill-name>.skill       # Compiled ZIP archive (derived artifact)
```

**SKILL.md** contains YAML frontmatter (`name`, `description`) followed by workflow instructions that tell the skill to detect the current phase and read the appropriate reference file in full.

**Phase A** — Interactive: the skill conducts a structured interview, produces 3–4 numbered markdown files (`s#-##-[client]-[description].md`) into an `ai_ticket_analysis/` folder.

**Phase B** — Report generation: reads the Phase A markdown outputs and produces a styled HTML report (Accenture-branded).

## The Three Sessions

| Session | Skill file | Input | Output files |
|---|---|---|---|
| 1 — Analysis | `aiops-analysis.skill` | Ticket data, SRE reports | `s1-01` through `s1-04` |
| 2 — Architecture | `aiops-architecture.skill` | `s1-01`, `s1-03`, `s1-04` | `s2-01` through `s2-04` |
| 3 — TEP (estimation) | `aiops-tep.skill` | Session 2 outputs | `s3-01` through `s3-03` |

## Key Conventions

**Output file naming:** `s#-##-[client]-[description].md`
- `#` = session number, `##` = sequential within session, `client` = variable placeholder for the engagement client name

**Output directory:** Always `ai_ticket_analysis/` (confirm with user before writing the first file).

**UC taxonomy:** 14 standard AIOps use cases (UC-01 through UC-14), defined in the analysis phase and referenced throughout architecture and estimation.

**Findings traceability:** Findings (`F-01`, `F-02`, …) discovered in Session 1 are explicitly tied to use cases in Session 2 architecture decisions.

**T-shirt sizing (Session 3):** Default conversion is `XS=1–2 pd, S=3–5 pd, M=6–10 pd, L=11–20 pd, XL=21–40 pd`. Users can override at the start of Session 3.

**Reference architecture lock:** `aiops-architecture/references/ref-agentic-architecture.md` is a locked pattern library (~40 000 lines). Do not edit it casually — it defines all architectural concerns (Compute, Data, Knowledge, ML Pipelines, Agent Orchestration, Memory, Integration, Observability, Reliability, Security, Governance, Quality/AgentOps, FinOps) across three deployment strategies (AWS-native, Azure-native, platform-independent).

## Packaging Skills

`.skill` files are ZIP archives of the skill source directory. Rebuild after editing source:

```powershell
# From AIOps/ directory — example for aiops-analysis
Compress-Archive -Path aiops-analysis/* -DestinationPath aiops-analysis.skill -Force
```

## Evaluating Skills

The `AIOps/aiops-analysis-workspace/` directory contains benchmarking data. Each `eval-N-*/` folder has:
- `without_skill/outputs/response.md` — baseline (no skill definition loaded)
- `with_skill/outputs/response.md` — response with skill active

Benchmark results are summarised in `iteration-1/benchmark.md` and `benchmark.json`. The current pass rate with skill active is ~87% vs ~35% without.

## Working with the skill-creator Skill

Use `/skill-creator` to create new skills, iterate on existing ones, or run evals. It understands the SKILL.md schema and the two-phase pattern used throughout this collection.
