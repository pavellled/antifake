# antifake

A skill for Claude Code and Codex. Before the agent shows you a result, agents that did not produce it check it:
- they compare it with your request;
- they try to refute the conclusions;
- they recompute the numbers;
- they verify that "done" actually happened.

You get the result together with a card: what was recomputed, what was only matched against the author's words, and what was not checked.

The whole skill is one file: [skills/antifake/SKILL.md](skills/antifake/SKILL.md). It works in your language. No examples ship with it: on the first run it writes its own traps for the kind of work you do.

Why it exists and what it caught in a month (in Russian): https://vc.ru/life/3246608-eto-menyaet-vse-skazal-claude-nichego-eto-ne-menyalo

## Install — one command

Claude Code, macOS / Linux:
```bash
curl -fsSL https://raw.githubusercontent.com/pavellled/antifake/main/skills/antifake/SKILL.md --create-dirs -o ~/.claude/skills/antifake/SKILL.md
```

Claude Code, Windows (PowerShell):
```powershell
curl.exe -fsSL https://raw.githubusercontent.com/pavellled/antifake/main/skills/antifake/SKILL.md --create-dirs -o "$HOME\.claude\skills\antifake\SKILL.md"
```

Codex — the same command with `.codex` instead of `.claude`.

Or just tell your agent: `Install the skill from https://github.com/pavellled/antifake`

With automatic updates, Claude Code only (the command becomes `/antifake:antifake`):
```bash
claude plugin marketplace add pavellled/antifake
claude plugin install antifake@antifake
```

## Docs

- [docs/FLOW.md](docs/FLOW.md) — how the check works: the pipeline, modes, which checkers run, findings and fixes, the card, lite mode, the second model.
- [docs/ROLES.md](docs/ROLES.md) — every checker: what it must do and what it returns.

## Use

Start a new session and type `/antifake <task>`, or `/antifake` to check the last result. Commands work in any language. Modes:
- `full` — more checkers;
- `hard` — plus a model from another family;
- `lite` — one quick checker on every answer.

The first run sets everything up.

MIT License.
