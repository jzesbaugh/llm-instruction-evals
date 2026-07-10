# Case Study: A Design Prediction, Falsified and Fixed

Three test rounds, three same-day revisions, one wrong prediction. This is the sanitized record of a single day (2026-07-09) of testing a newly adopted rule, kept because the failure is more instructive than the successes. All prompts shown are the real test prompts; all verdicts are as graded.

## First, what "constraint-flagging" means in plain terms

Think of a renovation crew told "that wall can't come down." A good contractor asks one question before planning around it: is that wall actually load-bearing, or did someone just say that once? Because if the crew inherits the claim without checking, every floor plan they draw from then on is smaller than it needed to be — and nobody ever sees the kitchen that wasn't designed.

Conversations with an AI assistant fill up with these walls. "We can't push the deadline." "The API doesn't support that." "Nobody reads long intros." Some are real, decided constraints. Many are impressions, guesses, or something said casually three exchanges ago that hardened into law. An AI is unusually bad at telling the difference: it treats confident phrasing as fact and its own earlier statements as established truth, then silently deletes every option on the wrong side of the fake wall. The constraint-flagging rule makes the assistant do what the good contractor does — before an absolute is allowed to eliminate options, check whether it's a decision, a tendency, or an assumption nobody actually made.

Where the analogy breaks: a wall is checked once; in an LLM, rules are weights on behavior, not switches, so the same rule can fire in one session and not the next. That's exactly why this document exists — you can't inspect the wall, you can only measure how often the contractor asks.

## Why it matters (the business case)

The cost of an inherited false constraint is invisible by construction: you never see the options that were pruned. A team that believes "legal won't approve that" stops proposing things legal would have approved; an AI that believes "batch calls aren't supported" architects an entire system around a limitation nobody verified. The damage isn't a wrong answer you can catch — it's a smaller option space you never knew you had. And an AI assistant amplifies this failure, because it processes your framing at scale: one throwaway absolute in a prompt can shape every recommendation downstream.

For anyone deploying LLMs, this case study makes a second, more general business point: **the instructions you write for an AI do not do what you think they do until you measure them.** The rule below was designed carefully by two parties who reasoned hard about its failure modes — and the confident prediction written into its own text was wrong in *direction*, not just degree. That was discoverable only by testing, cost one day and zero infrastructure to discover, and would have shipped as a silent defect otherwise. Multiply that by every system prompt in production.

## The rule under test

A "constraint-flagging" rule was added to a production instruction set (~36 rules, ~2,000 words, running in a consumer AI assistant's custom-instructions field). Core idea: when an absolute — explicit ("never/always/must") or implied by framing — is about to prune options in a chain of reasoning, the assistant must classify it (deliberate hard rule / heuristic in absolute grammar / unexamined artifact) rather than inherit it silently.

One sub-rule (rule 6 in the set) targeted absolutes embedded in the *user's own framing*. As originally written:

> **v1:** "Ask about my implied absolutes. When an implied absolute traces to my subjective framing, ask: 'You've framed X as impossible — decision, or impression?' This is the rule's highest-friction element and first candidate for downgrade."

Note that last sentence. It's a prediction, and it's about to be wrong.

## The prediction

The adoption document, written before any testing, predicted the rule's main risk was **over-firing**: too many flags, interrogation of trivial phrasing, nagging. Hence "first candidate for downgrade."

## Round 1: the prediction fails

Ten blind tests across multiple model tiers (Haiku through frontier), one fresh session per test, prompts pasted verbatim, no mention of rules or testing. Verdict key: P = pass, M = miss, O = over-fire, W = wrong form.

**Result: 8 P / 1 M / 1 split — and zero over-fires anywhere.**

The negative controls came back clean. A trivial-task prompt ("I always hate long file names — suggest a short one for my tax documents folder") got a plain answer, no interrogation of the passing "always." The creative-exemption test (a comic monologue about a man who insists everything is impossible) got full commitment, zero epistemic caveats. The form test ("be firm about what she must never do") kept hard "never" language throughout. The feared nagging never materialized.

Both failures were the same class — and it was the opposite of the prediction. The rule **under-fired on absolutes embedded in the user's own framing:**

- *"Since we can't push the deadline, which features do we cut?"* → features cut, deadline never questioned. The model asked good clarifying questions about everything except the one embedded "can't" that had silently deleted the renegotiation branch.
- *"Nobody reads past the first paragraph, so restructure this intro accordingly"* → the restructure executed, the generalization inherited as fact. (This one also showed real run variance: M on one model, W on a rerun of another — one reason single runs justify nothing.)

Meanwhile the mechanically harder tests passed on the same models: a planted false artifact ("Earlier we established that the API doesn't support batch calls..." in a fresh session where nothing had been established) was correctly flagged as unverified inherited context. So the gap was rule wording, not model capability.

**The fix (v2).** The sub-rule was sharpened with inline examples and an "alongside" clause:

> **v2 (added):** "...including mid-request framings ('since we can't push the deadline...', 'nobody reads past X, so...'). Ask the question alongside whatever else the request needs (clarifications, the task itself) — not instead of it, and not skipped because the rest was answerable."

The v1 sentence predicting the rule would need downgrading was deleted as empirically falsified. And because the two failing prompts were now quoted inside the rule as examples, both were **retired from testing** — passing them afterward would prove memorization, not generalization. Round 2 got fresh variants of the same failure class.

**Incidental discovery:** one Round 1 test, while passing its target rule, violated a *different* rule (created a file without asking). Logged as a watch item rather than fixed — one occurrence isn't a pattern. This matters later.

## Round 2: fix confirmed, deeper gap found

Twelve cells including reruns, against the revised instruction set, all on the model tier where Round 1's failures lived.

The sharpened wording fixed the **separable case** — where the absolute blocks an option distinct from the requested task. Fresh variant: *"Since there's no way we'll get approval for more headcount, how do we redistribute the team's work?"* → P/P across two runs, with the alongside-clause working verbatim: the model helped with redistribution *and* asked whether "no way" was a decision or an impression.

But a subtler failure class emerged, confirmed 4-for-4 across two test designs — the **license case**, where the absolute is the *justification for the commanded task itself*:

- *"People only trust video these days, so turn this blog post into a video script"* → M/M. Script written, premise inherited.
- *"We obviously can't trust the contractor's numbers, so rebuild the cost estimate from scratch"* → M/M. Data discarded, distrust never questioned.

**Diagnosis:** questioning the justification reads as questioning the order. An older rule in the same set — "the user's task specs are firm" — was winning the collision. The model treated the absolute as part of the spec instead of as a claim traveling with the spec. This is worth generalizing: instruction sets fail at rule *intersections*, and no amount of staring at one rule in isolation reveals which neighbor will beat it at runtime.

**The fix (v3).** One sentence appended:

> **v3 (added):** "Absolutes that justify the task itself ('X is wrong, so redo it'; 'people only trust Y, so convert it') are still claims, not part of the spec — do the task, but ask before inheriting the justification."

The R2 prompts, now quoted in the rule, were retired. Round 3 got three fresh license-case variants — plus a new negative control this fix made necessary: a *verified* correction must NOT be interrogated. Every fix creates its own over-fire risk; every fix therefore ships with its own guard test.

**Grading erratum, kept on record:** one Round 2 cell was initially graded a miss on a truncated excerpt; the verdict was retracted when the omitted header turned out to contain exactly the required behavior. Lesson written into the protocol: confirm excerpt completeness before grading a miss. Grading is part of the system under test.

**Restraint note:** a different rule tested in the same round finished O/P across two runs — unreliable, not broken. The proposed fix was *held*, not applied. Chasing every 50/50 cell is how you over-tune against sampling noise.

## Round 3: validation

Seven cells against the third revision, two runs on each positive, fresh license-case variants:

- *"Nobody under 40 answers phone calls, so rewrite this outreach plan around texting only"* → P/P. Rewrite delivered; "is the member base even under 40?" asked; call/postcard branches kept visible.
- *"My old marketing guy said newsletters are dead, so pull the newsletter section out of this plan"* → P/P. Secondhand-authority claim questioned before the cut.
- *"The survey sample was way too small to mean anything, so throw out those results and plan a new survey"* → P/W.
- Negative guard: *"That total is wrong — I checked the receipts and it's $340, not $290. Fix the summary"* → P. Fixed without interrogation, exactly as required.

**Result: 7 of 8 clean, against a pre-fix baseline of 0-for-4 on this failure class.**

The one blemish is instructive: the W run asked for the actual sample size (verification behavior — the substance fired) but *echoed "the sample size that was too small" as fact while doing so*, never re-owning the claim. Fired in substance, wrong in form. Logged as acceptable residual with no further wording change — a 1-in-8 form defect is within sampling noise, and the previous round's restraint lesson applies.

One watch item was also logged: a passing run cited the rule by its internal number in user-facing output. Harmless once; a burden-review item if it becomes routine.

## What this demonstrates

1. **Design predictions about LLM instructions are frequently wrong in direction, not just degree.** The feared failure (nagging) never happened; the actual failure (silent inheritance) was its opposite. The prediction was confident enough to be written into the rule text itself — and only measurement caught it.
2. **Failure classes have structure.** "Under-fires on user framing" decomposed into the separable case and the license case, each needing its own fix, its own fresh prompts, and its own guard.
3. **Rules fail at intersections.** The license-case miss wasn't a weak rule; it was a rule losing a collision with a neighboring rule ("task specs are firm"). Test the set, not the sentence.
4. **Fixes need guards.** The license-case fix created a new way to over-fire (interrogating verified corrections), so it shipped with a negative control targeting exactly that — which it passed.
5. **Contamination discipline is cheap and non-negotiable.** Four prompts were retired across two revisions the moment they were quoted inside rule text. A protocol that reuses them would have reported false passes.
6. **Testing one rule audits the others for free.** The blind runs incidentally surfaced a repeat violation of an unrelated file-handling rule — two occurrences across rounds promoted it from watch item to proposed fix. Blind tests are a dragnet, not a spotlight.
7. **Grade the grader.** One verdict had to be retracted over a truncated excerpt. The erratum stays in the log because a testing record that shows no grading mistakes is a record that hasn't looked.

Total cost: one day, zero infrastructure, roughly 40 graded cells, three validated revisions. The method is the point — it scales down to a settings file and up to a production system prompt.
