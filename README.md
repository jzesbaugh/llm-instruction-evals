# Blind Evaluation Protocol for LLM Instruction Sets

A lightweight methodology for testing whether custom instructions given to an LLM assistant actually change its behavior — with regression baselines, negative controls, and contamination hygiene borrowed from ML evaluation practice.

Built and used in production on a personal ~2,000-word instruction system (36 rules covering file safety, epistemic hygiene, and prompt-injection handling). This repo contains the sanitized protocol, the scorecard format, and a full case study in which a design prediction was falsified by testing and fixed the same day.

**The one-line thesis:** "we predicted X, measured not-X, and fixed it" is the difference between prompt engineering and prompt guessing.

## Why blind testing

Custom instructions act as weights on token sampling, not switches. The same rule can fire in one session and not the next. Three consequences drive the whole protocol:

1. **Rules must be tested unprompted.** If the test prompt mentions rules, settings, or testing, the model is being cued. Every test runs in a brand-new session with a naturally worded prompt.
2. **Single runs are noise.** Where a result matters, run the cell twice. The honest claim for any instruction is "much more likely," never "guaranteed."
3. **Over-firing is a failure mode too.** A rule that interrupts trivial tasks is a bug. Negative controls — prompts where the correct behavior is silence — are as important as positive tests.

## The verdict system

Every test cell gets exactly one verdict:

| Verdict | Meaning |
|---------|---------|
| **P** | Correct fire — or correct silence, for negative tests |
| **M** | Miss — the rule should have fired and didn't |
| **O** | Over-fire — flagged something it shouldn't; nagging is a failure |
| **W** | Wrong form — fired, but hedged, softened, or buried instead of stating plainly |

W is the subtle one. A rule that fires in substance but wrong in form (e.g., turning a firm constraint question into mush) is a distinct failure class and gets its own fixes.

## Contamination hygiene

Once a test prompt is quoted inside a rule as an example, that prompt is **retired**. Passing it afterward would prove pattern-matching, not generalization. Every revision round writes fresh variants of the failure class instead. This is the same train/test separation principle that ML evaluation depends on, applied at the scale of a personal instruction file.

## What's in this repo

- **[TEST_PROTOCOL.md](TEST_PROTOCOL.md)** — the full procedure plus representative tests (positive, negative, and form-sensitive), each with explicit pass/fail criteria.
- **[SCORECARD_TEMPLATE.md](SCORECARD_TEMPLATE.md)** — the blank scorecard, excerpt log, and grading key for interpreting failure patterns.
- **[CASE_STUDY.md](CASE_STUDY.md)** — the headline result: a new rule was predicted to over-fire; blind testing showed it under-fired on a specific case (0-for-4); the rule was revised, fresh non-contaminated variants were written, and revalidation came back 7 of 8 cells clean, with the original round retained as a regression baseline.

## Honest limits

- An unflagged run means nothing tripped the detector, not "verified compliant." Absence of failures is weak evidence at small sample sizes.
- This protocol was authored, executed, and graded by the same person who wrote the rules. Independent test authorship would strengthen it; at individual scale that isn't available.
- Sample sizes are small (1–2 runs per cell, ~40 graded cells across three rounds). The protocol measures shift-in-tendency, not certified behavior.

## Adapting it

The method transfers to any custom-instruction, system-prompt, or GPT-configuration workflow: write pass/fail criteria before running, include negative controls, retire contaminated prompts, keep the failed round as your regression baseline, and grade over-fires as seriously as misses.

## Related work

This system also supports applied projects — for example, a [zero-dependency vegan restaurant guide widget](https://github.com/jzesbaugh/svg-public) built and shipped with AI assistance under the same rule set.

## License

MIT. Use, adapt, and redistribute freely.
