# bloom

Learn an analysis an agent wrote — ML, causal, or statistical — one small mode at a time, climbing Bloom's taxonomy to mastery. Each invocation runs a single mode. Predictions come before reveals, and a quiz gates presenting.

## Install

Clone this repository, then copy the skill into Claude Code's skills directory:

```bash
git clone <this-repository-url> bloom_skills
cd bloom_skills
mkdir -p ~/.claude/skills/bloom
cp -R bloom/. ~/.claude/skills/bloom/
```

Cursor Agent Skills use the same `SKILL.md` format. For a personal Cursor skill, copy into `~/.cursor/skills/bloom/` the same way. For one repository only, copy into that repo's `.cursor/skills/bloom/`.

Open a new chat so the agent loads the skill. Invoke it by name; see the [root README](../README.md) for both skills and compatibility.

## Invocation

```text
/bloom diagnose
/bloom map
/bloom whatif
/bloom lab causal
/bloom sabotage
/bloom compare
/bloom arena
/bloom quiz
/bloom review --role vp
/bloom status
```

With no mode, or an unknown one, the agent prints the mode table and asks which to run. Useful options: `--chat` or `--html` to override format, `--n <k>` for item or round count, `--role <persona>` for review, `--no-roleplay` for due cards only, `--fresh` to rebuild the world model.

Run `diagnose` first. Later modes read `.bloom/world.json` from the project you are learning.

## Modes

| Mode | Level | Format | What you do |
|---|---|---|---|
| `diagnose` | Remember | chat | See the pipeline, estimand, headline result, and silent decisions |
| `map` | Understand | HTML | Walk a clickable pipeline graph; answer "why?" before the rationale unlocks |
| `whatif` | Apply | HTML | Score synthetic profiles; guess before the number shows |
| `lab [risk\|causal]` | Analyze | HTML | Drop features or change the adjustment set; predict before the result unlocks |
| `sabotage` | Analyze | chat | Find one secretly broken rerun from the outputs |
| `compare [node]` | Evaluate | chat | Pick between two methods before seeing results |
| `arena` | Evaluate | HTML | See whether the pipeline recovers a planted effect |
| `quiz` | Mastery gate | chat | Five items, one at a time; pass 4 of 5 before presenting |
| `review [--role r]` | Create | chat | Due cards, then a role-play (VP, PM, legal, and others) |
| `status` | | chat | Calibration, confident-wrong items, what's due, suggested next mode |

A typical climb is diagnose → map → whatif → lab → sabotage or compare → arena → quiz → review over the following days. The agent suggests one next mode; it does not chain modes on its own.

## Generated data and privacy

While you learn a project, the agent writes a `.bloom/` directory at that project's root (world model, precomputed results, HTML views, a local log server, `ledger.jsonl`, and `cards.json`). Those files are derived from the project. Add `.bloom/` to that project's `.gitignore`. This repository already ignores `.bloom/` so generated state is not committed here.

Views and chat reveals use aggregates, binned ranges, or synthetic profiles. The agent checks code excerpts for credentials, tokens, and absolute paths before embedding them. The view server binds to `127.0.0.1` and only trusts a port after `/whoami` matches this project.

When `~/.debt-ledger/ledger.jsonl` already exists (shared with [debt-check](../debt-check/README.md)), each ledger line is also appended there. That file lives in your home directory.
