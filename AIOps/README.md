# AIOps Engagement Skills

A set of [Claude Code](https://claude.com/claude-code) skills that deliver a structured, three-session **AIOps consulting engagement** — from raw operational data to a costed implementation plan.

The skills walk a client through analysis, architecture, and estimation. Each session produces working markdown files plus a polished, Accenture-branded HTML report, and each session's outputs feed the next.

## The three sessions

| # | Skill | What it does | Outputs |
|---|---|---|---|
| 1 | `aiops-analysis` | Discovers AIOps use cases from operational data (ticket exports, SRE reports, transcripts). | `s1-01`…`s1-04` + analysis report |
| 2 | `aiops-architecture` | Designs the target AIOps platform grounded in Session 1 findings — UC selection, layer/component design, governance and release plan. | `s2-01`…`s2-04` + architecture report |
| 3 | `aiops-tep` | Builds a task inventory, applies T-shirt sizing, and rolls up person-day estimates. | `s3-01`…`s3-03` + TEP report |

Every skill has two phases:

- **Phase A — Interactive.** A structured interview that produces numbered markdown working files.
- **Phase B — Report generation.** Transforms the Phase A markdown into a branded HTML report.

## Layout

```
AIOps/
├── aiops-analysis/         # Session 1 source (SKILL.md + references/)
├── aiops-architecture/     # Session 2 source (includes ref-agentic-architecture.md)
├── aiops-tep/              # Session 3 source
├── aiops-analysis.skill    # Packaged skill (ZIP)
├── aiops-architecture.skill
├── aiops-tep.skill
├── aiops-analysis-workspace/       # Evals + benchmark for Session 1
├── aiops-architecture-workspace/   # Evals for Session 2
└── aiops-tep-workspace/            # Evals for Session 3
```

## Evaluating the skills

Each skill has its own `*-workspace/` folder with an `evals/evals.json` defining test
cases (a Phase A start, a Phase B start, and an ambiguous start) and pass/fail assertions.
Running the evals produces per-run outputs and a rolled-up `benchmark.md` scorecard
comparing the skill's behaviour **with** vs **without** the skill loaded — the evidence
that the skill actually improves routing and adherence. Re-run after editing any
`SKILL.md` or reference prompt to catch regressions.

## Usage

Inside Claude Code, invoke a skill by intent — for example "let's start an AIOps engagement", "design the AIOps architecture", or "run the TEP session". The skills detect whether you're starting Phase A (interactive) or Phase B (report generation) and route accordingly.

All deliverables are written to an `ai_ticket_analysis/` folder (confirmed with you before the first file is created). The file-naming convention is `s#-##-[client]-[description]`, where `#` is the session number.

## Conventions

- **Use-case taxonomy** — 14 standard AIOps use cases (UC-01…UC-14) defined in Session 1 and carried through.
- **Findings traceability** — findings (`F-01`, `F-02`, …) from Session 1 are tied to architecture decisions in Session 2.
- **T-shirt sizing** — default conversion `XS = 1–2 · S = 3–5 · M = 6–10 · L = 11–20 · XL = 21–40` person-days (overridable in Session 3).
- **Reference architecture** — `aiops-architecture/references/ref-agentic-architecture.md` is a locked pattern library covering all architectural concerns across AWS-native, Azure-native, and platform-independent deployments.

## Developing these skills

See the repository [CLAUDE.md](../CLAUDE.md) for skill authoring guidance, the packaging command for rebuilding `.skill` archives, and the evaluation workflow.
