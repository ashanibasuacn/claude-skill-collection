# AIOps Architecture Session — Session 2 of 3

Great — that puts us squarely at the start of **Phase A: interactive architecture design**. I'll design the target AIOps platform for GlobalBank grounded in the Session 1 findings. The discipline I hold throughout: every component and pattern I recommend must trace back to an **F-## finding** — nothing goes in because it's "best practice."

We work through four phases, each producing one markdown file you save:

| Phase | Focus | Output file |
|---|---|---|
| 1 | Goals and constraints — strategic objective, existing landscape, autonomy ceiling | `s2-01-globalbank-arch-constraints.md` |
| 2 | UC selection and bundling — which use cases to build, in what groupings | `s2-02-globalbank-arch-uc-selection.md` |
| 3 | Platform decomposition — layers, components, technology choices | `s2-03-globalbank-arch-design.md` |
| 4 | AI/ML sequencing, governance, release plan, validation criteria | `s2-04-globalbank-arch-plan.md` |

I have your three Session 1 inputs to read before I design anything:

- `s1-01-globalbank-analysis-context.md` — strategic objective and customer vocabulary
- `s1-03-globalbank-analysis-findings.md` — the F-## findings register
- `s1-04-globalbank-analysis-uc-catalogue.md` — the candidate UC catalogue and tiering

I'll read those plus the agentic reference architecture (my pattern menu) first. **I haven't seen the file contents in this thread yet** — please paste or attach them so I can ground the design in the actual findings rather than assumptions. One check before we proceed: are the findings already in **F-## format**? If they're not, tell me and we'll decide whether to reformat them or run a best-effort reconstruction (I won't silently invent F-## numbers).

---

## Where we start: Phase 1 — Goals, constraints, and existing landscape

This is the architect's discovery phase, and it's a **mandatory pause point** — I won't design until you've answered. Once I've read the inputs, I'll confirm the strategic objective from Session 1 and then work through the questions below. You can answer them now alongside attaching the files if you'd like to move quickly.

**0. Output folder.** Default is `ai_ticket_analysis/` in the project root, with the `s2-NN-` order prefix on each file. Good to use that, or do you want a different path? I'll confirm before writing the first file.

**1. Strategic objective.** I'll restate the objective captured in Session 1 Phase 0 and ask you to confirm it still holds or tell me if priorities have shifted.

**2. Existing landscape.**
- ITSM platform (ServiceNow / Jira SM / BMC Helix / other)?
- Monitoring / observability stack — what tools, and what integrations already exist?
- Automation toolbox (Ansible, ServiceNow Workflows, other)?
- CMDB — present and accurate? Federated or single-source?
- Cloud platform (AWS / Azure / GCP / on-prem / hybrid)?
- APM / log aggregation in use?

**3. Constraints.**
- Data residency or cloud-region restrictions? (For a bank this is usually load-bearing.)
- Source-system access — read-only, read-write, or sandboxed?
- Existing licences or vendor commitments that must be preserved?
- Change-freeze windows that affect deployment?

**4. Risk surface.**
- Prohibited automated actions (e.g. no auto-close of P1s, no auto-restart of production CIs)?
- Regulatory audit-trail requirements?
- High-severity incident definition — what forces escalation to humans regardless of automation?

**5. Autonomy appetite.** Highest autonomy tier you're comfortable with for the **first release**:
- L1 — AI assists (recommendations only)
- L2 — AI acts on pre-approved action classes with human confirmation
- L3 — AI acts autonomously on low-risk tickets; human oversight on higher risk
- L4 — AI acts autonomously across most ticket types; exception-based oversight

**6. Governance ownership.** Named Knowledge Layer owner? Named governance authority? SRE maturity? Expected human-review SLAs?

---

To get moving: **attach the three s1 files** (and confirm they're in F-## format), and answer the Phase 1 questions when ready — starting with the output folder and the autonomy ceiling, since those shape everything downstream. Once I've read the inputs and have your answers, I'll produce `s2-01-globalbank-arch-constraints.md` and we'll pause there before moving to UC selection.
