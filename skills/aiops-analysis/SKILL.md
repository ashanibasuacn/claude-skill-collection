---
name: aiops-analysis
description: >
  Run the AIOps Analysis session — Session 1 of the 3-session AIOps engagement workflow. Use this skill whenever the user wants to start an AIOps engagement, analyze operational ticket data (IMS / AMS / SR), discover AIOps use cases, run an analysis session, or kick off AIOps discovery work for a client. Also invoke when the user has completed the analysis session and wants to generate the HTML analysis report from their session 1 markdown files. This skill covers both Phase A (interactive analysis, four-phase discovery) and Phase B (Accenture-branded HTML report generation). Always use this skill at the start of any AIOps engagement, even if the user just says "let's start AIOps" or "I have ticket data to analyse".
---

# AIOps Analysis — Session 1 of 3

This skill runs Session 1 of the AIOps engagement workflow. It has two phases:

- **Phase A** — Interactive analysis. Discovers AIOps use cases from the customer's operational data (ticket exports, SRE reports, transcripts). Produces four markdown files.
- **Phase B** — HTML report generation. Transforms those four markdown files into a polished, Accenture-branded report.

---

## Step 1 — Determine the phase

If the user's intent is already clear from context, go straight to the right phase. Otherwise ask:

> "Are you starting **Phase A** — the interactive analysis (you have operational data: ticket exports, SRE reports, workshop transcripts)?  
> Or **Phase B** — report generation (you have the four session 1 markdown files and want to produce the HTML report)?"

**Signals for Phase A:** mentions of ticket data, IMS / AMS / SR exports, transcripts, post-mortems, SRE reports, "start the engagement", "discover use cases".  
**Signals for Phase B:** mentions of `s1-01` through `s1-04` files, "generate the report", "analysis report", "ready for report".

---

## Phase A — Interactive Analysis

Read `references/phase-a.md` in full and follow every instruction in it exactly — role, UC taxonomy, interaction protocol, phase-by-phase structure, output templates, quality standards, and known limitations are all defined there.

**Output files** (all to `ai_ticket_analysis/` — confirm folder before the first file):

| File | Phase |
|---|---|
| `s1-01-[client]-analysis-context.md` | Phase 0 |
| `s1-02-[client]-analysis-baseline.md` | Phase 1 |
| `s1-03-[client]-analysis-findings.md` | Phase 2 |
| `s1-04-[client]-analysis-uc-catalogue.md` | Phase 3 |

**Carry forward to Session 2 (Architecture):** `s1-01`, `s1-03`, `s1-04`

---

## Phase B — HTML Report Generation

Read `references/phase-b.md` in full and follow every instruction in it exactly — confirmation questions, section plan, brand identity, technical requirements, and generation process are all defined there.

**Readiness detection:** The reference's session opening assumes the user still needs to attach files. If the user has already signalled readiness — e.g. "I have my four files ready", "s1-01 through s1-04 are done", or they paste file content directly — skip the "attach files and we'll start" step and go straight to the pre-generation confirmation questions in order: output folder → report format → client name → logo → date → confidentiality line → section plan.

**Input files needed:** `s1-01` through `s1-04` from Phase A  
**Output file:** `s1-05-[client]_analysis_report.html` (to `ai_ticket_analysis/`)

**After Phase B,** remind the user which files to bring into Session 2:  
`s1-01-[client]-analysis-context.md`, `s1-03-[client]-analysis-findings.md`, `s1-04-[client]-analysis-uc-catalogue.md`
