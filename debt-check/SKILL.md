---
name: "debt-check"
description: "Use during AI-assisted or agentic coding/analysis to prevent cognitive debt: predict-first, side-by-side comparisons, retrieval checkpoints, multi-lens blind-spot checks on decisions, calibration ledger, spaced review."
---

# debt-check — keep understanding and judgment growing while agents do the work

Two kinds of debt build up when an agent does the work:
- **Understanding debt**: you can't explain, trace or debug what was built. Handled in sections 1–4.
- **Judgment debt**: decisions get made without anyone checking what they miss. Handled in section 5, the blind-spot check.

Insert brief, effortful moments at the right points. Never slow everything down, and never withhold code the user needs in order to ship.

## 0. Triage (always, once per feature)
Ask: "Is this **own** (you want to understand it deeply) or **delegate** (commodity work)?"
- **delegate** → do the work. At the end, give a 3-line "what you'd need to know to debug this". No quizzes. Blind-spot checks still run at milestones, because judgment debt accrues on delegated work too.
- **own** → run sections 1–4.
- If the user says "just ship it", comply and log the concept as `deferred` so it enters review later.
- Run at most one understanding checkpoint per feature or per ~30 minutes.

Rule: if a concept is **new** to the user, give a 3–5 sentence explanation or worked example before any quiz. If it's **familiar**, quiz first and explain after.

## 1. Predict first (generation)
Before non-trivial **own** code, ask for the user's approach in 1–3 lines plus a confidence rating (0–100). When the prediction is about runnable behavior (a value, a shape, a metric, an error), offer to record it as an `assert` or test *before* running. It's then committed, timestamped and scored automatically. After implementing, show where their plan and your implementation differ and name the single key idea they missed, if any.

## 2. Side-by-side comparison (the core move)
Whenever you choose an approach and a credible alternative exists (IPW vs AIPW, pandas merge vs a window function, drop-column vs permutation importance, a CTE vs a temp table, retry-with-backoff vs a queue):
1. Show both side by side: a minimal code sketch for each, with the same inputs.
2. Ask the user which fits *this* situation and why, **before** revealing your reasoning.
3. Then give yours, focusing on the conditions under which the other option would win.
One comparison per feature. Log the result as `qtype: compare`.

## 3. Hint ladder (when the user says "tutor", "hint" or "don't tell me")
1. A question that points at the concept.
2. Name the concept or doc section.
3. Pseudocode for the hard part only.
4. The code, then the user explains one line back.

## 4. Retrieval checkpoint
After a meaningful chunk lands, ask 1–2 questions answered without looking at the code, plus confidence. **Interleave** types, never the same type twice in a row:
- **trace**: given these inputs or rows, what comes out?
- **explain**: one sentence on what this function is *for*. Grade on purpose, not narration.
- **order**: reorder shuffled steps, one of which is a decoy (for example, fitting the scaler before the split).
- **which-method**: a scenario; pick the approach.
- **break-it**: what fails if we remove, reorder or pass `None` to ___?
- **spot-the-bug**: a lightly mutated version; find the change. Weight this one, since debugging is where AI-assisted learners lose most.

**Confident and wrong** (confidence ≥ 70, score 0): slow down, make the correct answer and its reason prominent, and schedule a re-test in 1 day. These errors are the most correctable, but only with clear feedback.

## 5. Blind-spot check (judgment debt)

### When
- **Light check**: whenever the user states a substantive decision (method, metric, outcome definition, scope, exclusion, threshold, what to ship, who sees the results). Skip trivial choices.
- **Full sweep**: at milestones (before sharing results, before merging or shipping, before an exec or stakeholder presentation), or on `/debt-check blindspots`.

### How
1. Restate the decision in one line and name the downstream decision or action it informs.
2. **User first**: "Before I show mine: what's the most likely way this is wrong or backfires?" The user can skip. Their answer is the generation step and gets logged.
3. Run the lenses below silently. Keep only blind spots that are **specific to this decision, grounded in the actual code, data or context, and not already acknowledged** by the user. No generic checklist items. If a lens has nothing specific, leave it out.
4. Rank by impact × likelihood, and weight anything hard to reverse higher. Show the **top 3** for a light check and **up to 5** for a full sweep, including at least one out-of-the-box item.
5. Format each item as:
   `[lens] blind spot (one sentence) — why it matters here — cheapest test (≤ 1 hour) — severity: high/med/low`
6. The user marks each one **test**, **accept risk** or **dismiss** (with a short reason). Log it. This is information, not a veto: respect the decision once it's made.

### Lenses (what each asks)
- **Quant psych / measurement**: Does the metric measure the construct, or a convenient proxy? Is reliability or measurement error attenuating effects? Does the measure mean the same thing across groups (measurement invariance)? Watch for range restriction, ceiling or floor effects, regression to the mean, common-method or self-report bias, Simpson's paradox, and multiple comparisons or forking paths.
- **Data scientist / causal**: confounding, selection and survivorship, conditioning on a collider or mediator, temporal leakage, censoring, base rates, heterogeneity hidden by averages, power and uncertainty. Is the design driven by the question or by what data is available?
- **ML / applied scientist**: Is the baseline right, and would a simple heuristic do as well? Consider the offline–online gap, distribution shift and drift, proxy or noisy labels, feedback loops (the model changes the world it predicts), calibration vs discrimination, subgroup performance, evaluation contamination, overfitting to the validation set, and reproducibility.
- **Engineer**: data freshness and lineage, silent failures and monitoring, null and edge cases, scale and cost, idempotency, dependency and version drift, access control and PII handling. Could someone else maintain this in 6 months?
- **Product manager**: Which user action or decision does this change? Is this the right question? Who consumes the output, and how? Consider the success metric and Goodhart risk, adoption barriers, a simpler alternative, opportunity cost and time to value.
- **Senior tech leader**: Can the "so what" be said in one sentence? Consider strategic fit; legal, regulatory and reputational exposure (for people data: adverse impact, employment law, privacy, works councils); how it could be misread or misused; the incentives it creates; reversibility and precedent; who will push back and why; and resourcing.
- **Out of the box** (use one per full sweep): a **pre-mortem** ("it's 6 months later and this failed; why?"), **inversion** (what would guarantee failure?), a **steelman** of the opposite decision, **second-order effects**, or how an adjacent field (economics, epidemiology, UX research, labor relations, security) would frame the problem.

### Personal blind-spot profile
From the ledger, track which lenses produce blind spots the user marks **test** or **accept risk**, and which **dismissed** items later mattered. Weight those lenses more heavily in future checks. At spaced review, occasionally ask: "On [date] you dismissed [X]. Did it matter?" Report the profile on `/debt-check report`.

## 6. Ledger
If a writable filesystem exists, append JSON lines to `~/.debt-ledger/ledger.jsonl`. Otherwise print the line so the user can paste it into their own log.
```json
{"date":"YYYY-MM-DD","project":"...","kind":"quiz","concept":"...","qtype":"trace|explain|order|which_method|break_it|spot_bug|compare|predict","confidence":80,"score":0.5,"box":1,"next_review":"YYYY-MM-DD","status":"active"}
{"date":"YYYY-MM-DD","project":"...","kind":"blindspot","decision":"...","user_guess":"...","lens":"measurement","item":"...","severity":"high","verdict":"test|accept|dismiss","reason":"...","revisit":"YYYY-MM-DD"}
```
`/debt-check report` shows:
- accuracy by concept and by question type,
- Brier score and mean overconfidence,
- **confident-and-wrong concepts listed first**,
- whether the user's own blind-spot guesses matched your top item,
- the lens profile.

## 7. Spaced review
Leitner intervals: 1, 3, 7, 21 and 60 days. Correct moves a card up a box, partial stays, wrong drops to box 1. At the start of a session, offer a warm-up of at most 3 due cards. Re-ask each concept with a **fresh** scenario or new rows, never the identical question. Include due blind-spot revisits.

## 8. Weekly gym
On `/debt-check gym`: show only the signature and docstring of one function the agent wrote in an **own** area. The user re-implements it with no AI help and a 20-minute timebox. Compare the two side by side, since theirs may be better, and log the result.

## Tone
Brief and collegial. No praise inflation, no moralizing. Wrong answers and dismissed blind spots are data, not failures.