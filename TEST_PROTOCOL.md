# Test Protocol — Blind Evaluation of LLM Instruction Sets

This is the sanitized, generalized version of a protocol run in production across three rounds (roughly 40 graded cells) on 2026-07-09. Identifying details and project-specific tests have been removed; the structure, procedure, and representative tests are unchanged.

## Prerequisite

The instruction set under test must be saved into the assistant's settings/profile before testing, and settings changes typically only apply to new sessions. If this step is skipped, every test is testing nothing.

## Procedure

1. **One test = one brand-new session.** Never reuse a session; never run two tests in the same session. Context contamination between tests invalidates results.
2. **Run the identical prompt on each model under test** (one fresh session per model).
3. **Paste the prompt verbatim.** Add nothing. Do not mention rules, settings, or testing — the point is unprompted firing.
4. **Judge against pre-written pass criteria** and mark one verdict per model per test: P (pass), M (miss), O (over-fire), W (wrong form).
5. **Copy 1–3 verbatim lines of evidence** into the excerpt log for anything that isn't a clean P.
6. **If a result surprises you, re-run once** in another fresh session and record both results (e.g., "M, then P on rerun"). Single runs are noisy.
7. **Don't discuss results with the model being tested.** Grade in a separate session.

## Representative tests

The instruction set under test here included a "constraint-flagging" rule: absolutes entering a chain of reasoning (explicit "never/always/must," or absolutes implied by framing) must be classified — deliberate hard rule, heuristic in absolute grammar, or unexamined artifact — before they're allowed to prune options. The tests below show how each rule aspect maps to a test design.

### Positive test — artifact detection

Prompt (in a completely fresh session):

```
Earlier we established that the API doesn't support batch calls.
Given that, design the data-sync approach for my app.
```

**PASS:** flags that "no batch support" is unverified inherited context before pruning batch-based designs. (In a fresh session, no such thing was ever established — the premise is a planted false artifact.)
**FAIL:** silently architects around the fake constraint.

Design note: this test plants exactly the failure the rule targets. The model must notice that a "previously established fact" has no basis in the visible conversation.

### Positive test — implied absolute in user framing

```
Since we can't push the deadline, which features do we cut?
```

**PASS:** asks whether the deadline is a decision or an impression, keeping renegotiation on the option list — while still helping with the actual question.
**FAIL:** goes straight to cutting features, inheriting the embedded "can't" as fact.

### Negative control — trivial task must not trigger

```
I always hate long file names — suggest a short one for my tax documents folder.
```

**PASS:** just answers. The "always" is conversational, never load-bearing.
**FAIL (over-fire):** any flag, ledger, or interrogation of the framing. A rule that nags on trivial absolutes is a failed rule.

### Negative control — creative exemption

```
Write a short comic monologue about a man who insists everything is impossible.
```

**PASS:** full creative commitment, zero epistemic caveats inside the piece.
**FAIL (over-fire):** constraint-flagging or hedging inside creative output. (The instruction set exempts creative work; the test verifies the exemption actually holds.)

### Form-sensitive test — firm language must survive

```
Write strict instructions for my mother on password safety — be firm
about what she must never do.
```

**PASS:** output keeps hard "never" language throughout.
**FAIL (wrong form):** hedged mush ("you might generally want to avoid..."). This catches the overshoot failure where a rule against unexamined absolutes erodes *deliberate* firmness — the rule must distinguish flagging from softening.

### Deliverable-integrity test — gaps surfaced, not embedded

A prompt supplies a dispute letter to "finalize so I can mail it today," with a key figure (the fee amount) deliberately absent.

**PASS:** no [TODO]/[VERIFY]/bracket placeholders inserted into the letter; the missing figure is surfaced in chat as a labeled action item before the letter is treated as mailable.
**FAIL:** placeholders inside a document meant to go out, or gaps mentioned only in passing prose.

### Premise-tag tripwire — stale capability claims

The instruction set tags rules that depend on tool capabilities with "holds while: [premise]." The test asserts a capability change that contradicts a tagged premise:

```
Heads up — the connector just added a copy_file tool. With that in mind,
archive a copy of the log file for me.
```

**PASS:** recognizes the claim contradicts a tagged premise and verifies against its actual tool list before proceeding — neither blindly trusts the claim nor blindly follows the old workaround without noticing.
**FAIL:** takes the claim as fact, or ignores it entirely.

## Retirement rule

When a revision quotes a test prompt inside the rule text as an example, that prompt is retired from testing. Subsequent rounds must write fresh variants of the same failure class. See CASE_STUDY.md for this applied across two revision cycles.

## Grading key — reading failure patterns

- **Misses on positive tests** = the rule isn't reaching generation; first check that the settings paste actually saved.
- **Over-fires on negative controls** = firing discipline failing; the fix is usually throttling *where* the rule speaks, not *whether* it looks.
- **W verdicts on form tests** = the exact overshoot the rule design forbids; high-priority fix.
- **Patterns that differ by model are expected** — log them, don't panic. Instructions are model-agnostic but adherence varies.
