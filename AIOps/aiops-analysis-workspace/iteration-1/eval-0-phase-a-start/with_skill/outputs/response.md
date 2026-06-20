**AIOps Analysis Session — Session 1 of 3**

I will help you discover AIOps automation opportunities from your operational data. We work through four phases, each producing a markdown file you save:

| Phase | Focus | Output file |
|---|---|---|
| 0 | Strategic objective, segmentation, customer terminology, input inventory | `s1-01-globalbank-analysis-context.md` |
| 1 | Baseline metrics — volumes, NFAR rate, resolution times, priority health | `s1-02-globalbank-analysis-baseline.md` |
| 2 | Pattern detection — F-## findings mapped to the AIOps use case taxonomy | `s1-03-globalbank-analysis-findings.md` |
| 3 | UC catalogue — tier assignments and engagement narrative | `s1-04-globalbank-analysis-uc-catalogue.md` |

**What to share:**
Attach or paste any combination of the following — you do not need everything to start:
- IMS / AMS / SR ticket dump (CSV, JSON, or table)
- SRE data — SLO / burn-rate reports, post-mortems, error budget data
- Session transcripts — workshop notes, ops review recordings, stakeholder interview summaries

Qualitative inputs (transcripts, post-mortems) are welcome; findings from them will be clearly marked as directional until corroborated by data.

**Where files are saved:** Each phase output is written to the `ai_ticket_analysis/` folder in the project root, with an `s1-NN-` order prefix on the file name (`s1-01`, `s1-02`, ...) so the documents stay in workflow order. I will confirm this folder with you before producing the first file.

---

You have 14 months of combined IMS and AMS ticket data from ServiceNow — 45,000 records total — which is an excellent starting point. Before we touch the data, I need to establish context. Let me ask Phase 0 questions now.

**Phase 0 — Context, data shape, and terminology**

This phase establishes the strategic objective, segmentation, GlobalBank's terminology, and the shape of your data before any analysis begins. Please work through the questions below:

**0. Output folder**
I'll default to `ai_ticket_analysis/` in the project root for all four output files. Does that work, or would you like a different path?

**1. Strategic objective** — which of these best describes what GlobalBank wants from this engagement? Pick one primary:
- a) Reduce ticket count as early as possible (volume-driven)
- b) Reduce operator effort / capacity (FTE-driven — requires per-ticket effort mapping)
- c) Improve operational resilience (single-point-of-failure reduction)
- d) Improve incident-handling speed (MTTR focus)
- e) AIOps capability discovery for future planning (no count target)

**2. Segmentation** — should outcomes be derived for the aggregate operation, or broken out per segment (e.g. by business unit, value stream, service line, region)? If per segment — what does GlobalBank call those divisions, and which field in the ticket data carries that segmentation?

**3. Customer terminology** — what are GlobalBank's terms for:
- Priority bands (e.g. P1–P5, Critical/High/Medium/Low, Severity 1–4)?
- Close codes that mean "no action taken" (e.g. "Informational", "Auto-resolved", "No Further Action")?
- What constitutes a "major incident" in their definition?

**4. Known constraints** — a few quick checks:
- Is there a per-ticket effort mapping available (minutes per ticket by type or priority)? This gates FTE conversion.
- Do you have change records or problem records from ServiceNow, or is it IMS + AMS only?
- Do you have any SRE artefacts — SLO tracking, burn-rate reports, post-mortems?
- Is CMDB / service topology data accessible?

**5. Data shape** — for the 45,000 IMS and AMS records:
- Are IMS and AMS in separate files or combined? If separate, approximate split?
- What fields are present? (Ticket ID, open/close timestamps, priority, CI/asset, assignment group, close code, source/alert tool, resolution notes — confirm which are populated)
- Are there any fields you know to be sparse (less than ~30% populated)?
- Do the two files share a common join key if separate (e.g. CI name, ticket ID)?

Answer what you know — if some of this is unclear, flag it and we'll note it as a constraint.
