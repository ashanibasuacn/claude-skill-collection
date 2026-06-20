**AIOps Task, Estimation & Planning Session — Session 3 of 3**

Great — the architecture session is done, so we are starting **Phase A** of the TEP. I will decompose the GlobalBank architecture design into a structured task inventory, apply T-shirt sizing to each task, and convert those sizes into person-day estimates with low and high bounds.

We work through four phases, each producing a markdown file you save:

| Phase | Focus | Output file |
|---|---|---|
| 1 | Scope and assumptions — delivery releases, team model, estimation basis, T-shirt conversion table | `s3-01-globalbank-tep-assumptions.md` |
| 2 | Task inventory — one task per deliverable component, with category and owner role | `s3-02-globalbank-tep-tasks.md` |
| 3 | T-shirt sizing — XS / S / M / L / XL for every task | Updates `s3-02-globalbank-tep-tasks.md` |
| 4 | Person-day estimation — convert sizes to low/high ranges, roll up by bundle and release | `s3-03-globalbank-tep-estimates.md` |

**Default T-shirt conversion** (we will confirm or override this in Phase 1):
XS = 1–2 pd · S = 3–5 pd · M = 6–10 pd · L = 11–20 pd · XL = 21–40 pd

A few ground rules so the estimate stays defensible:
- Scope comes entirely from the architecture — I will not add components that aren't in the design, or drop ones that are.
- Every estimate rests on stated assumptions, and estimates are ranges (low / high), never single-point precision.
- This session produces **effort** estimates, not a project schedule — sprint sequencing and timeline depend on team capacity and are out of scope unless you give me capacity data.

**Where we start: the three Session 2 files you mentioned.** To build the inventory I will work from:
- `s2-03-globalbank-arch-design.md` — the components that become tasks
- `s2-04-globalbank-arch-plan.md` — release assignment (R1–R4) and gate conditions
- `s2-02-globalbank-arch-uc-selection.md` — the bundle grouping
- Optionally `s2-01-globalbank-arch-constraints.md` if you have it — for team model and risk surface

Please share those three files (paste them or point me to them in the repo). Once I can see them, we begin at **Phase 1 — Scope and assumptions**, which is a mandatory pause point: I will confirm a handful of things with you before any task is written or sized —

1. **Output folder** — default is `ai_ticket_analysis/` in the project root; tell me if you want a different path.
2. **Release scope** — which releases (R1 / R2 / R3 / R4) are priced vs. indicative only.
3. **Team model** — which roles are available and at what allocation.
4. **Estimation basis** — experienced team vs. new to platform, parallel vs. sequential work, and whether GlobalBank provides internal SMEs for knowledge transfer / runbook authoring.
5. **T-shirt conversion table** — confirm the defaults above or override them.
6. **Explicit exclusions** — what's out of scope for this estimate.

Go ahead and attach the s2-02, s2-03 and s2-04 files and we'll kick off Phase 1.
