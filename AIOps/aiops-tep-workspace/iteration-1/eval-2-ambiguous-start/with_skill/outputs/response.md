A work breakdown with effort estimates is exactly **Session 3 — Task, Estimation & Planning (TEP)**, the final session of the AIOps engagement workflow. What you've described — selected use cases, platform design, and a release plan — is precisely the Session 2 output this session consumes, so you're set up to start.

## AIOps Task, Estimation & Planning Session — Session 3 of 3

I will decompose the architecture design into a structured task inventory, apply T-shirt sizing to each task, and convert those sizes into person-day estimates with low and high bounds. We do **not** revisit the architecture decisions here — this session works *from* them. And to set expectations: the output is an **effort estimate** (person-days), not a project schedule. Sprint sequencing, resourcing, and timeline depend on team capacity and are out of scope unless you give me capacity data.

We work through four phases, each producing a markdown file you save:

| Phase | Focus | Output file |
|---|---|---|
| 1 | Scope and assumptions — releases in scope, team model, estimation basis, T-shirt conversion table | `s3-01-[client]-tep-assumptions.md` |
| 2 | Task inventory — one task per deliverable component, with category and owner role | `s3-02-[client]-tep-tasks.md` |
| 3 | T-shirt sizing — XS / S / M / L / XL for every task | Updates `s3-02-[client]-tep-tasks.md` |
| 4 | Person-day estimation — convert sizes to low/high ranges, roll up by bundle and release | `s3-03-[client]-tep-estimates.md` |

**Default T-shirt conversion** (we'll confirm or override this in Phase 1, before any sizing is converted):
XS = 1–2 pd · S = 3–5 pd · M = 6–10 pd · L = 11–20 pd · XL = 21–40 pd

We pause at the end of each phase so you can review. **Phase 1 is a mandatory pause point** — I won't size or estimate anything until you've confirmed scope, team model, and the conversion table.

## What to attach

So that every task traces back to a real component (no invented work, no missed components), please attach your Session 2 retail-banking outputs:

- `s2-03-[client]-arch-design.md` — the platform design (components → tasks)
- `s2-04-[client]-arch-plan.md` — the release plan (release assignment, gate-condition tasks)
- `s2-02-[client]-arch-uc-selection.md` — the selected use cases (bundle grouping)
- *(optional)* `s2-01-[client]-arch-constraints.md` — for team-model context and risk surface

If those files live under different names, just point me at them.

## Where files are saved

Each phase output is written to the `ai_ticket_analysis/` folder in the project root, with an `s3-NN-` order prefix so the documents stay in workflow order. I'll confirm this folder with you before producing the first file.

## How you'll run it

1. Attach the three Session 2 files above.
2. We start at **Phase 1** — I'll ask a focused set of questions, one at a time: which releases are in scope (commonly R1 and R2 priced, R3/R4 indicative), your team model and allocations, the estimation basis (experienced team vs. new to the platform; client SMEs for knowledge transfer or delivery team end-to-end), the conversion table, and explicit exclusions.
3. You confirm, I produce `s3-01`, then we move through Phases 2–4, pausing after each.
4. The three markdown files then feed the **TEP Report** step (Phase B), which turns them into a branded HTML report.

A couple of things worth knowing before we start:

- I can run a quick first pass on the substance: what's the **client name** I should use in the file names and headers, and roughly which **releases** are you expecting to price versus treat as indicative? Even a rough answer lets me tailor Phase 1.
- If you'd prefer to move quickly, you can tell me to *"run all phases continuously"* — but Phase 1 still stops for your sign-off, because every estimate downstream rests on those assumptions.

Attach the Session 2 files (and confirm the client name) and we'll begin Phase 1.
