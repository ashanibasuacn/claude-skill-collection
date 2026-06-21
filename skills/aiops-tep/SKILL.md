---
name: aiops-tep
description: >
  Run the AIOps Task, Estimation & Planning session — Session 3 of the 3-session AIOps engagement workflow. Use this skill when the user wants to build a task inventory, apply T-shirt sizing, produce person-day estimates, create a delivery estimate, or generate the TEP HTML report. Always invoke when the user says "TEP session", "Session 3", "task estimation", "T-shirt sizing", "person-days", "effort estimate", "work breakdown", "planning session", or "TEP report", even if they don't mention the word 'skill'. Also use when the user has completed the TEP analysis and wants to generate the report from their s3 markdown files.
---

# AIOps Task, Estimation & Planning — Session 3 of 3

This skill runs Session 3 of the AIOps engagement workflow. It has two phases:

- **Phase A** — Interactive TEP. Decomposes the architecture design into a task inventory, applies T-shirt sizing to each task, and converts sizes to person-day estimates with low and high bounds. Four phases covering assumptions, task inventory, sizing, and estimation roll-up. Produces three markdown files.
- **Phase B** — HTML report generation. Transforms those three markdown files into a polished, Accenture-branded TEP report.

---

## Step 1 — Determine the phase

If the user's intent is clear from context, go straight to the right phase. Otherwise ask:

> "Are you starting **Phase A** — the interactive TEP session (you have the Session 2 architecture files and are ready to build the task inventory and estimates)?  
> Or **Phase B** — report generation (you have the three Session 3 markdown files and want to produce the HTML report)?"

**Signals for Phase A:** mentions of Session 2 files (`s2-02`, `s2-03`, `s2-04`), architecture design, "build the tasks", "estimate the work", "T-shirt sizing", "person days", "start TEP".  
**Signals for Phase B:** mentions of `s3-01` through `s3-03` files, "generate the TEP report", "TEP HTML", "ready for report".

---

## Phase A — Interactive TEP

Read `references/phase-a.md` in full and follow every instruction in it exactly — role, prerequisites, interaction protocol, phase-by-phase structure, output templates, T-shirt conversion table, and quality standards are all defined there.

**Output files** (all to `ai_ticket_analysis/` — confirm folder before the first file):

| File | Phase |
|---|---|
| `s3-01-[client]-tep-assumptions.md` | Phase 1 |
| `s3-02-[client]-tep-tasks.md` | Phases 2 & 3 |
| `s3-03-[client]-tep-estimates.md` | Phase 4 |

**Default T-shirt conversion** (confirm or override in Phase 1):  
`XS = 1–2 pd · S = 3–5 pd · M = 6–10 pd · L = 11–20 pd · XL = 21–40 pd`

**Inputs needed from Session 2:** `s2-02-[client]-arch-uc-selection.md`, `s2-03-[client]-arch-design.md`, `s2-04-[client]-arch-plan.md`; optionally `s2-01-[client]-arch-constraints.md` for team model and risk surface

---

## Phase B — HTML Report Generation

Read `references/phase-b.md` in full and follow every instruction in it exactly.

**Readiness detection:** The reference's session opening assumes the user still needs to attach files. If the user has already signalled readiness — e.g. "I have my s3 files ready", "s3-01 through s3-03 are done", or they paste file content directly — skip the "attach files and we'll start" step and go straight to the pre-generation confirmation questions in order: output folder → report format → client name → logo → date → confidentiality line → section plan.

**Input files needed:** `s3-01`, `s3-02`, `s3-03` from Phase A; optionally `s2-04` for release context  
**Output file:** `s3-04-[client]_tep_report.html` (to `ai_ticket_analysis/`)

**After Phase B,** the engagement is complete. All six deliverables are in `ai_ticket_analysis/`:  
`s1-05` (analysis report), `s2-05` (architecture report), `s3-04` (TEP report), plus the supporting markdown files.
