# debt-check

Keep understanding and judgment from going stale while an agent does the work. Understanding debt is when you cannot explain, trace, or debug what was built. Judgment debt is when a decision ships without anyone naming what it misses. The skill inserts short, effortful checkpoints and still delivers the code you need to ship.

## Install

Clone this repository, then copy the skill into Claude Code's skills directory:

```bash
git clone <this-repository-url> bloom_skills
cd bloom_skills
mkdir -p ~/.claude/skills/debt-check
cp -R debt-check/. ~/.claude/skills/debt-check/
```

Cursor Agent Skills use the same `SKILL.md` format. For a personal Cursor skill, copy into `~/.cursor/skills/debt-check/` the same way. For one repository only, copy into that repo's `.cursor/skills/debt-check/`.

Open a new chat so the agent loads the skill. It also applies during AI-assisted coding and analysis when the description matches. See the [root README](../README.md) for both skills.

## Usage

Once per feature, the agent asks whether the work is **own** (you want to understand it) or **delegate** (commodity work). Delegate work still gets a three-line debug note and milestone blind-spot checks. Own work uses the checkpoints below, at most one understanding checkpoint per feature or about 30 minutes. Saying "just ship it" skips the quiz and logs the concept as deferred for later review.

| Moment | What happens |
|---|---|
| Predict first | Before non-trivial own code, you state an approach and a confidence (0–100). Runnable predictions can be recorded as a test before the run. |
| Side-by-side | When a credible alternative exists, you see both sketches and pick before the agent's reasoning. One comparison per feature. |
| Hint ladder | Say "tutor", "hint", or "don't tell me" for a question, then a concept name, then pseudocode, then the code. |
| Retrieval | After a chunk lands, one or two questions without looking at the code, with confidence. |
| Blind spots | On a substantive decision, a light check. At share, merge, or presentation time, a full sweep. |
| Spaced review | Up to three due cards at the start of a session, each with a fresh scenario. |
| Weekly gym | You re-implement one agent-written function from its signature, with a 20-minute timebox. |

```text
/debt-check blindspots
/debt-check report
/debt-check gym
```

A light blind-spot check restates the decision, asks how it could be wrong, then shows the top three specific risks. A full sweep shows up to five. You mark each item **test**, **accept risk**, or **dismiss**. That record informs later checks; it does not block the decision.

## Ledger and privacy

If the filesystem is writable, the agent appends JSON lines to `~/.debt-ledger/ledger.jsonl` in your home directory. Lines record the date, project name, concept, question type, confidence, score, and review schedule, plus blind-spot decisions, lenses, severity, and your verdict. They do not belong in the project repository.

If that path cannot be written, the agent prints the JSON line so you can store it yourself.

`/debt-check report` summarizes accuracy, Brier score, overconfidence, confident-and-wrong concepts, whether your blind-spot guess matched the top item, and which lenses you tend to test, accept, or dismiss. The [bloom](../bloom/README.md) skill appends its own ledger lines to this same file when it already exists.
