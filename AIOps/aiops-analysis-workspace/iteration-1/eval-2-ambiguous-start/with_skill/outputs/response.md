# Response

**AIOps Analysis Session — Session 1 of 3**

I will help you discover AIOps automation opportunities from your operational data. We work through four phases, each producing a markdown file you save:

| Phase | Focus | Output file |
|---|---|---|
| 0 | Strategic objective, segmentation, customer terminology, input inventory | `s1-01-[client]-analysis-context.md` |
| 1 | Baseline metrics — volumes, NFAR rate, resolution times, priority health | `s1-02-[client]-analysis-baseline.md` |
| 2 | Pattern detection — F-## findings mapped to the AIOps use case taxonomy | `s1-03-[client]-analysis-findings.md` |
| 3 | UC catalogue — tier assignments and engagement narrative | `s1-04-[client]-analysis-uc-catalogue.md` |

**What to share:**
Attach or paste any combination of the following — you do not need everything to start:
- IMS / AMS / SR ticket dump (CSV, JSON, or table)
- SRE data — SLO / burn-rate reports, post-mortems, error budget data
- Session transcripts — workshop notes, ops review recordings, stakeholder interview summaries

Qualitative inputs (transcripts, post-mortems) are welcome; findings from them will be clearly marked as directional until corroborated by data.

**Where files are saved:** Each phase output is written to the `ai_ticket_analysis/` folder in the project root, with an `s1-NN-` order prefix on the file name (`s1-01`, `s1-02`, ...) so the documents stay in workflow order. I will confirm this folder with you before producing the first file.

Share what you have and we will begin.

---

You have IMS tickets from the last year and ops workshop transcripts — that is a solid starting point. The IMS ticket dump will give us quantitative findings; the transcripts will provide directional signals that we will clearly label throughout.

A note on what is gated with your current inputs: without AMS, SR data, change records, or problem records, some use cases (UC-15 change management, UC-16 problem lifecycle, UC-03 alert suppression depth, UC-22 FinOps) will be partially or fully gated. We will name these explicitly in Phase 3.

**Before we go to Phase 0**, one question to orient us:

What have you been able to get hold of in terms of the ticket export? Specifically:
- Roughly how many IMS ticket records, and what date range do they cover?
- What fields are included (e.g. ticket ID, open/close timestamps, priority, CI, assignment group, resolution notes, close code, alert source)?
- Are the transcripts from a discovery workshop, an ops review, or stakeholder interviews — and do you have them as text you can paste or attach?

Once I have that picture, we will move straight into Phase 0.
