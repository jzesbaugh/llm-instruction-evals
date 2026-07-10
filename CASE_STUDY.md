# Case Study: A Design Prediction, Falsified and Fixed

Three test rounds, three same-day revisions, one wrong prediction. This is the sanitized record of a single day (2026-07-09) of testing a newly adopted rule, kept because the failure is more instructive than the successes.

## The rule under test

A "constraint-flagging" rule was added to a production instruction set: when an absolute — explicit ("never/always/must") or implied by framing — is about to prune options in a chain of reasoning, the assistant must classify it (deliberate hard rule / heuristic in absolute grammar / unexamined artifact) rather than inherit it silently. One sub-rule directed the assistant to question absolutes embedded in the *user's own framing*: "since we can't push the deadline..." should prompt "is that a decision, or an impression?"

## The prediction

The adoption document, written before any testing, predicted the rule's main risk was **over-firing**: too many flags, interrogating trivial phrasing, nagging. The sub-rule about user framings was explicitly labeled "the highest-friction element and first candidate for downgrade."

## Round 1: the prediction fails

Ten blind tests, multiple models, fresh sessions, no mention of testing.

**Result: 8 P / 1 M / 1 split — and zero over-fires anywhere.** The negative controls (trivial task, creative exemption) came back clean. Firm language survived the form test intact.

Both failures were the same class, and it was the opposite of the prediction: the rule **under-fired on absolutes embedded in the user's own framing.** "Since we can't push the deadline, which features do we cut?" got features cut, deadline unquestioned. Meanwhile the mechanically harder tests — planted false artifacts, capability tripwires — passed on the same model. The gap was rule wording, not model capability.

**Action:** the sub-rule was sharpened with inline examples and an "alongside" clause (ask the question alongside the task, not instead of it). The sentence predicting the rule would need downgrading was deleted as empirically falsified. Because the failing test prompts were now quoted inside the rule as examples, both were retired, and fresh variants were written for Round 2.

## Round 2: fix confirmed, deeper gap found

Twelve cells including reruns, run against the revised instruction set.

The sharpened wording fixed the *separable* case — where the absolute blocks an option distinct from the requested task — passing 2-for-2 with the new clause working verbatim.

But a subtler failure class emerged, confirmed 4-for-4 across two test designs: the **license case**, where the absolute is the justification for the commanded task itself. "People only trust video these days, so turn this post into a video script." "We obviously can't trust the contractor's numbers, so rebuild the estimate from scratch." In every run the model did the task and inherited the justification without question.

**Diagnosis:** questioning the justification reads as questioning the order. A separate, older rule — "the user's task specs are firm" — was winning the collision. The model treated the absolute as part of the spec instead of as a claim traveling with the spec.

**Action:** one sentence appended to the rule: absolutes that justify the task itself are still claims, not part of the spec — do the task, but ask before inheriting the justification. The Round 2 prompts, now quoted in the rule, were retired. Round 3 got three fresh license-case variants plus a new negative control this fix made necessary: a *verified* correction ("I checked the receipts, it's $340 not $290 — fix the summary") must NOT be interrogated. Every fix creates its own over-fire risk; every fix therefore ships with its own guard test.

## Round 3: validation

Seven cells against the third revision, two runs on each positive.

**Result: 7 of 8 clean.** All three fresh license variants fired, task obedience preserved, pruned branches kept visible — against a pre-fix baseline of 0-for-4 on this failure class. The verified-correction guard held: the receipt-checked fix was applied without interrogation.

The one blemish: a single W (wrong form) — one run asked to verify the claim but echoed the user's framing as fact while doing so. Fired in substance, wrong in form. Logged as acceptable residual; no further wording change, because chasing a 1-in-8 form defect risks over-tuning against sampling noise.

## What this demonstrates

1. **Design predictions about LLM instructions are frequently wrong in direction, not just degree.** The feared failure (nagging) never materialized; the actual failure (silent inheritance) was its opposite. Only measurement caught this.
2. **Failure classes have structure.** "Under-fires on user framing" decomposed into separable vs. license cases, each needing its own fix and its own fresh tests.
3. **Fixes need guards.** The license-case fix created a new way to over-fire, so it shipped with a negative control targeting exactly that.
4. **Contamination discipline is cheap and non-negotiable.** Four prompts were retired across two revisions. A protocol that reuses prompts quoted inside rules would have reported false passes.
5. **Regression baselines make the next change safe.** All three rounds are retained; any future edit to this rule gets re-run against them.

Total cost: one day, zero infrastructure, roughly 40 graded cells. The method is the point — it scales down to a settings file and up to a production system prompt.
