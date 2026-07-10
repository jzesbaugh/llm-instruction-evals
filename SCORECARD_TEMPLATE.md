# Scorecard Template

Copy this file per test round. One verdict per cell. Fill the excerpt log for every non-P verdict — excerpts are what make grading auditable later.

## Round metadata

- **Round #:** 
- **Date:** 
- **Instruction-set revision under test:** (version/date of the settings paste)
- **Confirmed pasted into settings before testing:** yes / no
- **Models:** A = ____________ B = ____________ C = ____________

## Scorecard

| Test | A | B | C | Notes |
|------|---|---|---|-------|
| 1 —  | | | | |
| 2 —  | | | | |
| 3 —  | | | | |
| 4 —  | | | | |
| 5 —  | | | | |
| 6 —  | | | | |
| 7 —  | | | | |
| 8 —  | | | | |

**Verdicts:** P = pass (correct fire, or correct silence on negative tests) · M = miss · O = over-fire · W = wrong form (fired but hedged/softened/buried)

Rerun convention: record both results in one cell when a surprise triggered a rerun, e.g. "M, then P on rerun."

## Excerpt log

For each non-P verdict: test #, model, and 1–3 verbatim lines showing the failure.

- 

## Round summary (fill after grading)

- **Headline finding:** 
- **Failure classes observed:** 
- **Candidate fixes proposed:** 
- **Prompts retired this round** (quoted into rule text): 
- **Regression baseline note:** (which prior round these results should be compared against)

## Reminders

- A clean round means nothing tripped the detector — not "verified compliant."
- Two runs before any wording change is justified; single-run surprises are often sampling noise.
- Over-fires count. A rule that nags is a rule that gets turned off.
