# bloom skills

Two agent skills for learning analyses and for keeping your own understanding current while an agent writes code. Each skill is a directory with a `SKILL.md` file. Claude Code and Cursor both load that format.

| Skill | What it is for |
|---|---|
| [bloom](bloom/README.md) | Learn an AI-written ML, causal, or statistical analysis one mode at a time, from diagnose through quiz and review. |
| [debt-check](debt-check/README.md) | Predict first, compare alternatives, retrieve, and check blind spots so understanding and judgment keep up with the agent. |

They are meant to be used together. Bloom suggests `/debt-check blindspots` when a decision is worth challenging. When `~/.debt-ledger/ledger.jsonl` exists, bloom appends its practice log to that same file.

## Repository layout

```text
bloom/SKILL.md          # skill instructions (install this directory)
bloom/README.md         # how to install and run bloom
debt-check/SKILL.md     # skill instructions (install this directory)
debt-check/README.md    # how to install and run debt-check
```

`SKILL.md` is the file the agent reads. The READMEs are for people.

## Install

Clone this repository, then copy each skill directory into your skills folder:

```bash
git clone <this-repository-url> bloom_skills
cd bloom_skills
mkdir -p ~/.claude/skills/bloom ~/.claude/skills/debt-check
cp -R bloom/. ~/.claude/skills/bloom/
cp -R debt-check/. ~/.claude/skills/debt-check/
```

Cursor personal skills live under `~/.cursor/skills/`. Project skills live under `.cursor/skills/` in the repository where you want them. Copy the same two directories there if you want Cursor to load them from its own skill path. Details and invocation examples are in [bloom/README.md](bloom/README.md) and [debt-check/README.md](debt-check/README.md).

Open a new chat after copying so the agent picks up the new skills.

## Compatibility

Both files use the shared skill frontmatter (`name` and `description`) plus a markdown body. That layout works in:

- **Claude Code**, from `~/.claude/skills/<name>/SKILL.md`
- **Cursor**, from `~/.cursor/skills/<name>/SKILL.md` (all projects) or `.cursor/skills/<name>/SKILL.md` (one repository)

Invoke them by name:

```text
/bloom diagnose
/bloom status
/debt-check blindspots
/debt-check report
```

Bloom runs exactly one mode per invocation. Debt-check applies during agentic coding and analysis, and on the commands above.
