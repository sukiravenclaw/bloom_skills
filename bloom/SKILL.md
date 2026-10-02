---
name: "bloom"
description: Learn an AI-written analysis (ML, causal, stats) one small mode at a time, climbing Bloom's taxonomy to mastery. Run /bloom diagnose, map, compare, lab, whatif, arena, sabotage, quiz, review (VP/PM/legal role-play) or status.
---

# bloom: one mode at a time, up the taxonomy to mastery

Named after Benjamin Bloom, and built on two of his ideas:

- **The taxonomy.** Each mode works one level of Bloom's revised taxonomy: remember, understand, apply, analyze, evaluate, create.
- **Mastery learning.** The quiz is a gate you pass before presenting. Every miss gets corrective feedback and a retest on a fresh item, until you reach mastery.

The review role-plays stand in for Bloom's one-to-one tutor, the "2 sigma" condition.

The user wants to understand an analysis an agent wrote or helped write, without being handed everything at once. **Run exactly one mode per invocation.** Never build a full dashboard, never chain into the next mode on your own, and end every mode with one line suggesting a next mode.

Match the user's stated expertise. Don't dumb things down; do surface what the AI decided silently.

## Usage

`/bloom <mode> [target] [options]`

| Mode | Bloom level | Format | What it does | Time |
|---|---|---|---|---|
| `diagnose` | Remember | chat | Read the project, build the world model, show what the code decided | 3 min |
| `map` | Understand | HTML | The pipeline as a clickable graph; answer "why?" before the rationale unlocks | 5 min |
| `whatif` | Apply | HTML | Live model on synthetic profiles; guess the score before it shows | 5 min |
| `lab [risk\|causal]` | Analyze | HTML | Drop features or change the adjustment set; predict before the result unlocks | 10 min |
| `sabotage` | Analyze | chat | One secretly broken rerun; find the bug from the outputs | 5 min |
| `compare [node]` | Evaluate | chat | One decision, two methods side by side; pick before seeing results | 3 min |
| `arena` | Evaluate | HTML | Simulated worlds with a planted true effect; does the pipeline recover it? | 5 min |
| `quiz` | Mastery gate | chat | Five items, one at a time; pass 4 of 5 before presenting | 5 min |
| `review [--role r]` | Create | chat | Due cards, then a role-play (VP, PM, legal, skeptic…) where you build and defend the story; scorecard | 10 min |
| `status` | | chat | Calibration, confident-wrong list, what's due, the highest level reached per skill, suggested next mode | 1 min |

Options: `--chat` or `--html` to override a mode's format (HTML modes can run in chat with tables; chat modes stay in chat). `--n <k>` sets the number of items or rounds. `--role vp|pm|legal|ml|eng|methods|newhire|manager|panel|random` (or any described persona). `--no-roleplay` makes review run only the due cards. `--fresh` rebuilds the world model.

With no mode, or an unknown one, print the table and ask which mode to run.

## Shared workspace: `.bloom/` at the project root

| File | Written by | Holds |
|---|---|---|
| `world.json` | diagnose | Pipeline nodes, decisions, alternatives, silent decisions, concept novelty, estimand, results, source fingerprint |
| `precompute.py` | the first mode that needs numbers | Imports the project's own code and computes one mode's numbers: `python .bloom/precompute.py <mode>` → `results/<mode>.json` |
| `results/<mode>.json` | precompute.py | Every number a view or reveal shows |
| `views/<mode>.html` | map, lab, whatif, arena | One small page per mode |
| `serve.py` | the first HTML mode | Local server that serves the views and logs events (code below) |
| `ledger.jsonl` | every mode, and the server | One line per prediction, answer, compare, sabotage round or role-play score |
| `cards.json` | quiz, review, and anything that grades | Leitner state per skill: `{skill: {box, due, right, wrong, last_item, hyper}}` |

Rules:

- If `world.json` is missing, run `diagnose` first and say so in one line. If it's stale (the fingerprint of the source files changed), list the changed nodes and ask whether to re-diagnose; prioritize those nodes in the next quiz or review.
- Also append every ledger line to `~/.debt-ledger/ledger.jsonl` when that file exists (it's shared with the debt-check skill).
- Suggest adding `.bloom/` to `.gitignore` once. Results are derived from the user's data.

Ledger line shape (add fields as needed):

```json
{"ts":"…","mode":"lab","skill":"collider","item":"adj:+retention_bonus","pred":"down 1.0pp","conf":70,"actual":-0.011,"score":1,"source":"chat|view"}
```

## Rules for every mode

1. **Compute, never fake.** Every number comes from running the project's code, through `precompute.py`. If something can't be computed, say so; never estimate a number for a reveal.
2. **Predict before reveal.**
   - In chat, ask the question and **stop**. The answer must never appear in the same message as its question.
   - In HTML, keep the result hidden until the user commits a prediction.
   - Ask for confidence (0–100) with every prediction.
3. **Confident and wrong** (confidence ≥ 70, score 0): say so plainly, give the correct reasoning prominently, set that skill's card to box 1, due tomorrow, with `hyper: true`.
4. **Novelty order.** For a concept that's new to the user, give 3–5 sentences of explanation first, then ask. For a familiar one, ask first and explain after.
5. **Small outputs.**
   - Chat turns stay under ~150 words, except for reveals.
   - Each HTML view does one job, fits in about one screen, uses no framework, embeds its JSON, uses system fonts so it works offline, supports light and dark, and needs no build step.
6. **Privacy.** No row-level records of real people in any view or chat reveal; use aggregates, binned ranges or synthetic archetypes. Check code excerpts for credentials, tokens and absolute paths before embedding them.
7. **Blind spots** belong to the debt-check skill. If a mode surfaces a decision worth challenging, suggest `/debt-check blindspots` in one line; don't run it.
8. **Finish** with one result line and `Next: /bloom <mode>`, chosen from the ledger: the weakest skill, anything due, or the next mode in the sequence below.

Suggested sequence, not enforced, climbing the taxonomy: diagnose → map → whatif → lab → sabotage or compare → arena → quiz → review over the following days.

## HTML views: how to build and open

- Write `views/<mode>.html` with the data from `results/<mode>.json` embedded as a JSON literal. Escape `</` as `<\/` inside the script.
- Wrap all page code in an IIFE, so short names can't collide with anything else on the page.
- Record every commit with `log(event)`. When served, it POSTs to `/log`; opened as a file, it keeps events in localStorage and shows a **Copy results** button whose JSON the user can paste back into chat for you to append to the ledger.

  ```js
  const log = ev => { ev.mode = MODE; fetch('/log',{method:'POST',body:JSON.stringify(ev)}).catch(()=>{ try{const k='bloom-'+MODE; const a=JSON.parse(localStorage.getItem(k)||'[]'); a.push(ev); localStorage.setItem(k, JSON.stringify(a));}catch(e){} }); };
  ```

- Serve and open, but **verify before you trust a port** — a 200 on a shared port may belong to a *different* project's `.bloom`. Protocol:
  1. If `.bloom/serverinfo.json` exists, read its `url`+`root`; `GET <url>whoami` and use that server **only if** `root` equals this project's root. A dead/mismatched/foreign server fails this check.
  2. Otherwise start `python .bloom/serve.py` in the background (it picks a per-project port, skips busy ports, and writes `serverinfo.json`), then read the URL back from `serverinfo.json` (or the printed line) and confirm `/whoami` before using it.
  3. Open `<url>views/<mode>.html` with `python -m webbrowser <url>`. If no server can start, open the file directly.
  Never open a bare `http://127.0.0.1:8765/...` on faith — always resolve the URL through `serverinfo.json` + `/whoami`.
- Before handing it over, check once that every number traces to `results/<mode>.json`, results really stay locked until commit, the page works at phone width, and no row-level personal data is embedded.
- Tell the user the URL in one line. Don't describe the page.
- Layout conventions: horizontal left-to-right flow for pipelines; rounded frames grouping each stage; at most 2–3 semantic colors; the vendor name on data-source nodes (Workday, Greenhouse, Snowflake); tabular numbers.

`serve.py` (write it verbatim the first time an HTML mode runs):

```python
"""Serve .bloom/views and log what you do in them.  python .bloom/serve.py [--port N]

The default port is DERIVED from this project's path, so two different projects never
pick the same port by accident. If the chosen port is busy, the next free one is used.
The live URL + owning project root are written to .bloom/serverinfo.json on start (and
removed on clean exit), and `GET /whoami` returns the project root — so a client can
verify it reached the RIGHT project's server before trusting a 200."""
import argparse, datetime as dt, hashlib, http.server, json, pathlib
HERE = pathlib.Path(__file__).resolve().parent
ROOT = str(HERE.parent)                       # the project that owns this .bloom
LEDGER = HERE / "ledger.jsonl"
DEBT_LEDGER = pathlib.Path.home() / ".debt-ledger" / "ledger.jsonl"
INFO = HERE / "serverinfo.json"
PORT_LO, PORT_HI = 8765, 8965                 # scan range

def default_port():
    h = int(hashlib.sha256(ROOT.encode()).hexdigest(), 16)
    return PORT_LO + (h % (PORT_HI - PORT_LO))

class Handler(http.server.SimpleHTTPRequestHandler):
    def __init__(self, *a, **kw):
        super().__init__(*a, directory=str(HERE), **kw)
    def do_GET(self):
        if self.path == "/whoami":
            body = json.dumps({"bloom": True, "root": ROOT}).encode()
            self.send_response(200); self.send_header("Content-Type", "application/json")
            self.send_header("Content-Length", str(len(body))); self.end_headers(); self.wfile.write(body); return
        if self.path in ("/", "/index.html"):
            views = sorted((HERE / "views").glob("*.html")) if (HERE / "views").exists() else []
            links = "".join(f'<li><a href="/views/{v.name}">{v.stem}</a></li>' for v in views) or "<li>No views yet.</li>"
            body = f"<!doctype html><meta charset=utf-8><title>Bloom views · {ROOT}</title><body style='font:16px system-ui;margin:40px'><h1>Bloom views</h1><p style='color:#888'>{ROOT}</p><ul>{links}</ul>".encode()
            self.send_response(200); self.send_header("Content-Type", "text/html; charset=utf-8")
            self.send_header("Content-Length", str(len(body))); self.end_headers(); self.wfile.write(body); return
        super().do_GET()
    def do_POST(self):
        if self.path != "/log":
            self.send_error(404); return
        try:
            n = int(self.headers.get("Content-Length", 0))
            event = json.loads(self.rfile.read(min(n, 200_000)) or b"{}")
            if not isinstance(event, dict): raise ValueError("event must be a JSON object")
        except Exception as e:
            self.send_error(400, str(e)); return
        event.setdefault("ts", dt.datetime.now().isoformat(timespec="seconds")); event.setdefault("source", "view")
        line = json.dumps(event, ensure_ascii=False) + "\n"
        with LEDGER.open("a", encoding="utf-8") as f: f.write(line)
        if DEBT_LEDGER.exists():
            with DEBT_LEDGER.open("a", encoding="utf-8") as f: f.write(line)
        self.send_response(204); self.end_headers()
    def log_message(self, fmt, *args):
        if "POST /log" not in (str(args[0]) if args else ""): super().log_message(fmt, *args)

def bind(preferred):
    for port in [preferred] + [p for p in range(PORT_LO, PORT_HI) if p != preferred]:
        try:
            return http.server.ThreadingHTTPServer(("127.0.0.1", port), Handler), port
        except OSError:
            continue
    raise SystemExit(f"no free port in {PORT_LO}..{PORT_HI}")

if __name__ == "__main__":
    ap = argparse.ArgumentParser(); ap.add_argument("--port", type=int, default=None)
    srv, port = bind(ap.parse_args().port or default_port())
    url = f"http://127.0.0.1:{port}/"
    INFO.write_text(json.dumps({"port": port, "url": url, "root": ROOT,
        "started": dt.datetime.now().isoformat(timespec="seconds")}, indent=2))
    print(f"Bloom views for {ROOT} at {url}  (Ctrl+C to stop)")
    try:
        srv.serve_forever()
    finally:
        try:
            if INFO.exists() and json.loads(INFO.read_text()).get("port") == port: INFO.unlink()
        except Exception: pass
```

---

## Mode: diagnose (chat)

1. Find the entry points (notebooks, `main`, pipeline scripts) and read them. Classify the task as predictive, causal or descriptive. For a large project, build the world around the **one** model or estimate that matters most, and list the others as future worlds.
2. Write `world.json`:
   - **nodes**: ordered stages from source to output. For each: its purpose in one sentence, a ≤ 25-line code excerpt with file:line, input/output shapes, the decision made there, 1–2 credible alternatives, concept novelty (new or familiar), and skill tags.
   - the features or covariates, the outcome or treatment, and the estimand with its identification assumptions,
   - the headline results with their uncertainty,
   - **silent decisions**: thresholds, splits, defaults, dropped rows, imputation, class weights, and outcome definitions the user never asked for,
   - a fingerprint (a hash of each source file),
   - a **skill list**, a Q-matrix used to tag every item and card: for example leakage, estimand, collider, mediator, positivity, measurement, calibration, split design, API semantics.
3. Reply with at most 10 lines: the pipeline in one line, the estimand, the headline result, and the silent decisions as a **numbered** list.
4. Ask two things, then stop:
   - "Which numbers were news to you?"
   - "Which one decision most changes the result, and how confident are you (0–100)?"
5. After the answers, log them, run a quick check (rerun with that one decision flipped) to reveal which decision matters most, and suggest the first mode:
   - many surprises → `map`
   - a causal project → `lab causal`, then `arena`
   - a confident miss → `compare` on that decision.

## Mode: map (HTML)

The view is the pipeline graph only.

- Clicking a node opens an inspector with its purpose, the code excerpt, the decision and its alternatives.
- For familiar nodes, a "Why this choice?" textbox must be answered (or skipped) before the rationale shows. New nodes show their rationale first.
- A side list of silent decisions with checkboxes labelled "I didn't know this"; each tick is logged.
- A progress count of nodes opened.

No metrics, charts or quizzes on this page.

## Mode: compare [node] (chat)

1. Pick the node given as an argument. Otherwise pick the highest-impact decision not yet compared (from the ledger).
2. Show two minimal code sketches (≤ 8 lines each) with the same inputs: the pipeline's choice and one credible alternative. For three-way choices (g-comp, IPW, AIPW), show all three.
3. Ask which fits this situation and why, with confidence. **Stop.**
4. After the answer, run both options through `precompute.py compare <node>` and show one results table.
5. Then give 2–3 sentences on the conditions under which the other option wins. Log the result; tag the skill.

One comparison per invocation.

## Mode: lab [risk|causal] (HTML)

Choose from the task type if no argument is given. Precompute with `precompute.py lab`:

- **risk**:
  - drop-one ablations with the spread across CV folds,
  - drop-group ablations (the user's own feature groups),
  - pairwise ablations for the top 6, plus the additive expectation,
  - permutation importance for contrast, labelled as such.
- **causal**:
  - the estimate under the recommended adjustment set,
  - each confounder removed,
  - no adjustment,
  - the mediator added, the collider added, and both added (only when those roles exist in the DAG).
- For every change, one precomputed sentence of reason, derived from the real numbers.

View:

- Toggles on the left. For causal, a small DAG where the changed node is highlighted.
- A prediction form: direction, a magnitude slider and a confidence slider. **The result stays locked until they commit.**
- The reveal is a number line with the guess and the actual value, plus the interval for causal changes.
- If the guess missed by more than a threshold, an explain-back textbox, then the precomputed reason.
- A running table of the user's predictions and their Brier score.

## Mode: whatif (HTML)

- **Model.** Export the model for live JavaScript scoring if it's linear, logistic, a GAM or a small tree; otherwise precompute a grid of profiles.
- **Profiles.** 3–4 synthetic archetype profiles, never real people.
- **Guess mode (default on).** After any change, the score stays hidden until the user types a guess, which is logged with its absolute error. A toggle turns guess mode off for free exploration.
- **Drivers.** Contribution bars for the current profile, and a small ICE line for the slider being moved.
- **Subgroups.** A collapsed table of calibration and error rates by relevant subgroup, hidden until the user predicts which subgroup is worst calibrated.
- **Surprise me.** Jumps to the 3 least intuitive behaviours, precomputed and explained in one sentence each: an interaction the model can't represent, a missing value that hides risk, correlated features splitting credit.

## Mode: arena (HTML)

Only for causal or estimation projects. For a purely predictive project, say so and suggest `sabotage`.

- **Worlds.** In `precompute.py arena`, simulate 2–3 worlds that mirror the data's structure (marginals, correlations, censoring), each with a **planted effect**:
  1. one where the assumptions hold,
  2. one with an unmeasured confounder,
  3. one with a violation specific to this design: positivity, parallel trends or informative censoring.
- **Estimates.** Run the real pipeline on each world and store the estimate, interval, raw difference, effective sample size and propensity range.
- **World cards.** One card per world showing only its description and the observable clues. The user predicts whether the interval covers the truth, the direction of the error and their confidence. Then a number line reveals the truth, the estimate and the raw difference.
- **Heterogeneity.** An optional question: which subgroup has the largest true effect?
- **Diagnostics** for the real data, below the cards and shown after the first world is revealed: propensity overlap histogram, covariate balance before and after weighting, the E-value (or robustness value), and any negative-control result. Note that they only cover measured covariates.
- **No spoilers.** Never print the truths or estimates in the terminal or in chat before the user commits.

## Mode: sabotage (chat)

One round per invocation (`--n` for more).

1. Secretly pick one realistic bug that fits this project:
   - target leakage,
   - adjusting for a collider or a mediator,
   - a random split where time order matters,
   - complete-case dropping of non-responders,
   - balanced class weights,
   - a join that duplicates rows,
   - a filter that silently drops a subgroup,
   - a unit or scale error,
   - a stale snapshot date.

   Actually run the broken variant.
2. Show a baseline-vs-rerun table (n, base rate, key metric, calibration, estimate, top 3 features) with the changed cells marked. Mask any column name that would give the answer away.
3. Ask what broke. On a wrong guess, give one hint at a time: first what to look at, then which stage, then the family of bug. Never give the answer unless the user asks to reveal it.
4. After the user solves it or gives up, explain how to catch this bug in real work, and offer to add that check as a real test or assertion in their repo.

## Mode: quiz (chat)

1. Build 5 items from `world.json` and the results. Interleave the types, never the same type twice in a row:
   - **trace**: given these rows, what does this step output?
   - **explain**: what is this function for, in one sentence?
   - **order**: reorder shuffled steps, one of which is a decoy,
   - **which-method**,
   - **compare**: when would the alternative win?
   - **two-tier**: an answer, then its reason chosen from misconception-based options,
   - **spot-the-mutation**.

   Distractors come from the project's real silent decisions.
2. Present **one item at a time**, with a confidence prompt. **Stop and wait.**
3. After each answer, give a verdict and the reasoning in ≤ 80 words.
   - Grade free-text answers 0, 0.5 or 1 against a rubric you fix before asking.
   - Update that skill's card.
4. At the end, give the score:
   - 4/5 → "cleared to present";
   - otherwise, list the skills to review.

   Log the attempt. A second pass at least 7 days later earns "owner".

## Mode: review [--role r] (chat)

**Part 1: due cards.** Up to 5 cards whose `due` date has passed, in Leitner boxes 1/3/7/21/60 days. Each card gets a **fresh item** for the same skill (new rows, a new scenario, new wording), never a repeat of `last_item`. Grade each card the same way as in quiz. Skip this part if nothing is due or the user asks. With `--no-roleplay`, stop after this part.

**Part 2: role-play.** Use the persona from `--role`. If none is given, pick the persona that tests the user's weakest skills.

| Persona | What they probe | How they push |
|---|---|---|
| `vp` | The so-what in one sentence, the decision it changes, confidence, the cost of being wrong | Interrupts jargon; wants a number and a recommendation |
| `pm` | Which user action changes, the success metric, Goodhart risk, the timeline, a simpler alternative | "What would we do differently on Monday?" |
| `legal` | Adverse impact, protected groups, consent and data use, explaining it to employees, retention | Asks for documentation; flags risky wording (role-play only, not legal advice) |
| `ml` | Baseline, leakage, calibration versus ranking, drift, evaluation contamination | Asks for the one experiment that would change your mind |
| `eng` | Data freshness, failure modes, monitoring, reproducibility, cost | "What pages someone at 2am?" |
| `methods` | Construct validity, measurement invariance, identification, sensitivity analysis | Reviewer 2 energy, fair but relentless |
| `newhire` | Can you explain it plainly? | Keeps asking "why?" until the explanation is simple (Feynman test) |
| `manager` | A manager whose team the model flags | Pushes back emotionally; tests how you handle uncertainty and fairness |
| `panel` | The user gives a 60-second readout; three personas each ask one question | |

Any other `--role` text (for example `--role "CHRO focused on fairness"`) is played as described, under the same rules.

Rules:

- Stay in character for 3–5 exchanges. One question per turn. No flattery.
- Push on the weakest point of the last answer.
- Every challenge must come from this project's actual facts, results and silent decisions. No generic questions.
- Then step out of character for a debrief:
  - a scorecard from 1 to 5 on clarity, correctness, handling uncertainty, "so what", and knowing when to concede versus defend;
  - the one answer to redo, with a model answer of ≤ 80 words;
  - new cards for the concepts missed.
- Log the scores.

## Mode: status (chat)

Summarize from the ledger and cards:

- the modes done, and when,
- Brier score and mean overconfidence,
- the confident-and-wrong items, listed first,
- the highest Bloom level reached per skill (a level counts once a mode at that level is scored ≥ 0.7 on that skill),
- skills by mastery (the Beta(1+right, 1+wrong) mean; once there are 30+ graded responses, fit a DINA model on the Q-matrix instead and say which skills it flags),
- the cards due today,
- one suggested next mode.

Keep it to 10 lines or fewer.
