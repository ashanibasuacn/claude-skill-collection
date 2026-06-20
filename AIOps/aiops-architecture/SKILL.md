---
name: aiops-architecture
description: >
  Run the AIOps Architecture session — Session 2 of the 3-session AIOps engagement workflow. Use this skill when the user wants to design the AIOps target architecture, run architecture session 2, select use cases for implementation, decompose the platform into layers and components, design governance and release plans, or generate the architecture HTML report. Always invoke when the user says "architecture session", "Session 2", "design the AIOps platform", "UC selection", "release plan", or "architecture report", even if they don't use the word 'skill'. Also use when the user has completed Phase A and wants to generate the architecture report from their s2 markdown files.
---

# AIOps Architecture — Session 2 of 3

This skill runs Session 2 of the AIOps engagement workflow. It has two phases:

- **Phase A** — Interactive architecture. Designs the target AIOps platform grounded in the Session 1 findings. Four phases covering constraints, UC selection, platform design, and release/governance plan. Produces four markdown files.
- **Phase B** — HTML report generation. Transforms those four markdown files into a polished, Accenture-branded architecture report.

---

## Step 1 — Determine the phase

If the user's intent is clear from context, go straight to the right phase. Otherwise ask:

> "Are you starting **Phase A** — the interactive architecture design (you have the Session 1 analysis files and are ready to design the platform)?  
> Or **Phase B** — report generation (you have the four Session 2 markdown files and want to produce the HTML report)?"

**Signals for Phase A:** mentions of Session 1 files (`s1-01`, `s1-03`, `s1-04`), analysis findings, "design the architecture", "select use cases", "what platform", F-## findings, "start architecture".  
**Signals for Phase B:** mentions of `s2-01` through `s2-04` files, "generate the architecture report", "architecture HTML", "ready for report".

---

## Phase A — Interactive Architecture Design

Read `references/phase-a.md` in full and follow every instruction in it exactly — role, prerequisites, interaction protocol, phase-by-phase structure, output templates, and quality standards are all defined there.

**Also read `references/ref-agentic-architecture.md`** — this is the pattern library for AIOps systems. The Phase A instructions tell you to read it as a menu of options, not a template to copy.

**Output files** (all to `ai_ticket_analysis/` — confirm folder before the first file):

| File | Phase |
|---|---|
| `s2-01-[client]-arch-constraints.md` | Phase 1 |
| `s2-02-[client]-arch-uc-selection.md` | Phase 2 |
| `s2-03-[client]-arch-design.md` | Phase 3 |
| `s2-04-[client]-arch-plan.md` | Phase 4 |

**Inputs needed from Session 1:** `s1-01-[client]-analysis-context.md`, `s1-03-[client]-analysis-findings.md`, `s1-04-[client]-analysis-uc-catalogue.md`

**Carry forward to Session 3 (TEP):** `s2-02`, `s2-03`, `s2-04` (and optionally `s2-01` for team model and risk surface)

---

## Phase B — HTML Report Generation

Read `references/phase-b.md` in full and follow every instruction in it exactly.

**Readiness detection:** The reference's session opening assumes the user still needs to attach files. If the user has already signalled readiness — e.g. "I have my s2 files ready", "s2-01 through s2-04 are done", or they paste file content directly — skip the "attach files and we'll start" step and go straight to the pre-generation confirmation questions in order: output folder → report format → client name → logo → date → confidentiality line → section plan.

**Input files needed:** `s2-01` through `s2-04` from Phase A; optionally `s1-03` for F-## traceability chips  
**Output file:** `s2-05-[client]_architecture_report.html` (to `ai_ticket_analysis/`)

**After Phase B,** remind the user which files to bring into Session 3:  
`s2-02-[client]-arch-uc-selection.md`, `s2-03-[client]-arch-design.md`, `s2-04-[client]-arch-plan.md`
