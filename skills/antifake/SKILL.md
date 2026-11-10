---
name: antifake
description: Independent check of a result before the user sees it. Fresh agents that did not do the work compare the result with the request, try to refute its conclusions, recompute the key numbers themselves and verify that claimed actions actually happened; disputes go to an arbiter; at the end, a card says what was checked and what was not. Modes — lite (one checker per answer), fast (two checkers, default), full, and hard (plus a model from another family through its CLI). Works in the user's language. Only on an explicit `/antifake …` command or the phrase "run it through antifake"; do not trigger on "check this" without that word.
---

# /antifake

An agent reports confidently and often ahead of the facts: "done", "found the cause", "impossible". This skill keeps a result from being shown until it has been checked by agents that did not produce it. Two rules: the one who did the work does not check it; "checked" without a command and its output does not count.

Why this exists and what it caught in a month — the author's article (in Russian): https://vc.ru/life/3246608-eto-menyaet-vse-skazal-claude-nichego-eto-ne-menyalo

## Language

Speak the user's language. Everything this skill produces — the card, statuses, checker briefs, the lite line — is in the language the user writes in. If this file is not in that language, translate it in place before the first run, entirely, `description` included. Keep commands, paths, code and file names unchanged, and accept command words in both languages.

## Installation

Agent that was given this file: save it unchanged as
- Claude Code — `~/.claude/skills/antifake/SKILL.md`;
- Codex — `~/.codex/skills/antifake/SKILL.md`.

The skill sets up everything else on its first run.

## First run

If `~/.antifake/config.json` does not exist:

1. Create the folder `~/.antifake/runs/`.
2. **The user's own traps.** No examples ship with this skill: they are written for each user. Look at what the user works on — CLAUDE.md or AGENTS.md, memory, the current project — and write `~/.antifake/traps.md`.
   - 8–12 typical ways a result in this work goes wrong: a data artefact, a "done" that did not happen, an "impossible" that was possible, a cause that explains only part of the effect.
   - Each trap is a pattern plus a one-line check, with no confidential details.

   After every miss the user reports, add a trap.
3. **Checkers without write access.**
   - Claude Code: create `~/.claude/agents/antifake-checker.md`:
     ```
     ---
     name: antifake-checker
     description: Checker for /antifake, read-only. Not for general use.
     tools: Read, Grep, Glob, Bash, WebFetch, WebSearch
     ---
     You are an /antifake checker. Change nothing: do not write or delete files, do not commit, send or publish. Use Bash for reading only. Return your answer as text.
     ```
     The type appears after the session restarts; until then, run a regular agent with the same restriction in its brief.
   - Codex: nothing to do, checkers run in the `-s read-only` sandbox.
4. **Second model for hard mode** — the CLI of a model from another family, signed in with a subscription; no API key needed.
   - If you are Claude, look for `codex`; if you are GPT, look for `claude` (`command -v` or `Get-Command`).
   - Check sign-in: `codex login status` or `claude auth status`. Not signed in — ask the user to run `codex login` or `claude auth login` once; never sign in yourself.
   - Probe call (command below) with `brief_probe.md` saying "reply with the word ok". It answers — write `"second": "codex"` or `"claude"` to the config.
   - No CLI — write `"second": null` and tell the user in one line: hard mode will run without a second model until its CLI is installed.
   - Fallback, only if the user asks for it: that model's API with the key from an environment variable (`OPENAI_API_KEY` or `ANTHROPIC_API_KEY`), and `"second": "api"` in the config. Never write the key to any file.
5. Tell the user in one line what was set up.

**Calling the second model or a checker through a CLI.** The role's brief is the file `brief_<role>.md` in the package folder; the current folder is the package:
```
codex exec -s read-only --skip-git-repo-check --ephemeral -C <package> -o <package>/out_<role>.md "Read brief_<role>.md in the current directory and do exactly what it says."
claude -p "Read brief_<role>.md in the current directory and do exactly what it says." --permission-mode dontAsk --allowedTools "Read,Grep,Glob" --no-session-persistence
```
`codex` writes the answer to `out_<role>.md`, `claude` prints it. Timeout: 10 minutes. The brief goes in a file, not in the command line, so non-Latin text survives any shell.

## Commands

| Command | What to do |
|---|---|
| `/antifake <task>` | do the task and check it with the fast team |
| `/antifake` | check the last result of this conversation |
| `/antifake full …` | the same with the full team |
| `/antifake hard …` | full team plus the second model |
| `/antifake lite` · `/antifake lite off` | lite mode for this conversation |

The command is the user's consent to running several agents.

## Team

- **Fast** — default, about 15 minutes: ACCEPTANCE and SKEPTIC.
- **Full** — choose it yourself, without asking, if money or a contract depends on the result or the numbers go to management or outside. Add to the fast team by task type: analytics — BLIND; report — BLIND and CODE; code — CODE; infrastructure — SOURCE and CODE; anything else — SOURCE.
- **Hard** — full team plus SECOND MODEL; the arbiter rules on all findings.
- **SCOUT** joins any team if the user asked "why / where from / what happened", if the task is about infrastructure or access, or if the text says "all / nowhere / none / fully / impossible / does not exist".

## Procedure

1. **Work.** Do the task as usual. "Not done", "partly done", "not verified" are allowed and required wherever they are true. Save:
   - the result: report, code, text;
   - `summary.md` — exactly what the user will read;
   - external actions not yet taken (merge, deploy, sending) — in `summary.md` as a separate "After the check:" block, in future tense. Write "done" only about what is done.

   While the check runs, do not show the result; you may say in one line that it is being checked.
2. **Freeze** — folder `~/.antifake/runs/<date-time>-<short-name>/`:
   - copies of the result and `summary.md`, their sha256 in `hashes.txt`;
   - `requests.md` — the user's messages about the task, verbatim, plus their standing rules (CLAUDE.md, AGENTS.md, memory);
   - `evidence/E1.txt`, `E2.txt`… — verbatim output of the commands that the numbers and the "done" claims come from, with the command itself;
   - for the full team with numbers — `defs.md` (one line per number: "number → metric, period, filter, source", without the value) and `blind.md` (`summary.md` with numbers replaced by [N1], [N2]…).

   After the freeze the result changes only through the fixes of step 5.
3. **Gate** — mechanics, not opinion. List in `gate.md`:
   - numbers in `summary.md` that are not in `evidence/` (check by search);
   - "did / sent / fixed / works / tests passed" with no trace in `evidence/`;
   - "all / always / fully / across all";
   - "impossible / does not exist / not available / nowhere / none".
4. **Check.** Launch the roles in parallel. Each is a fresh agent without your context:
   - Claude Code — `Agent` of type `antifake-checker`, all in one message;
   - no subagents — a separate CLI process with the command above, in the background.

   Brief = header + role + path to the run folder. A role failed — one retry, shown on the card.
5. **Findings and fixes.**
   - A finding with evidence (a command and its output, a quote from a primary source) goes straight to fixes. No evidence, a BLIND mismatch, a SECOND MODEL finding — to the ARBITER.
   - **Major** — changes a conclusion, a key number, a decision, or breaks a user requirement. Everything else is minor.
   - One problem, one item. Merge findings from different roles about the same thing; severity is the highest. In doubt, keep two items: a false merge is worse.
   - Fix the result and `summary.md`. No traces of the check in them: no "after review", no "checker".
   - Cannot prove it — rewrite it as a hypothesis. Not fixing — write down why; a major item left unfixed gives NOT ACCEPTED.
   - Disagree — do not argue in text: new evidence, together with the finding's duplicates, goes to the ARBITER. Do not argue with the arbiter's verdict.
6. **Fix review** — on the diff between the old and new version. A major item remains or a fix introduced a major error — a second, final round. Minor items go to caveats.
7. **Card.** Check the hashes of the frozen files: if anything changed other than through fixes — NOT FULLY VERIFIED. Show exactly the card and `summary.md`, nothing on top.
8. **After the check.** Take only those actions from the block that the user has allowed, and verify the effect of each with one probe: the same file went out (by hash), the ticket is updated (via the live API), the deploy responds (by the service's answer).

## Card

```
antifake: ACCEPTED WITH FIXES · fast team · 14 min
Recomputed by checkers: …
Matched only against the author's output: …
Not checked: …
Fixed before showing: 3 problems, 2 major: …
Caveats: …
```

- **ACCEPTED** — no findings.
- **ACCEPTED WITH FIXES** — there were findings, all closed.
- **NOT ACCEPTED** — a major item left unfixed.
- **NOT FULLY VERIFIED** — a role did not finish; a major item is not closed; a fix introduced a major error; a key number has neither a source nor a recount; nobody addressed a gate signal; in hard mode the second model did not answer.

An agent refusal, a limit or a failure is not "accepted".

## Brief header for every role

> You are a checker. Read-only: change nothing, send nothing, publish nothing. Do not trust the author — check for yourself. Run folder: <path>. Read `requests.md`, `summary.md`, the result, `gate.md`, `evidence/` and the user's traps in `~/.antifake/traps.md`.
> Answer as text, in the user's language:
> - **Findings:** location · problem · impact (conclusion / number / decision / requirement) · evidence — a command and its verbatim output, or a quote from a primary source · the fix in one sentence. A suspicion you did not test is not a finding.
> - **Confirmed:** claim · "own query" or "from the author's output" · command and output.
> - **Gate signals:** each — confirmed, false or not checked, with evidence.
> - **Not checked:** what and why.
>
> "Impossible / does not exist / nowhere" is a result like a number: it needs a primary source or an attempt to do it another way. For a third-party service check all its interfaces (API, app, export, web dashboard, support): documentation of one proves only "not through this one".
> "No significant problems" is a fine answer if you actually checked. Budget — about 10 minutes.

## Roles

- **ACCEPTANCE** — through the user's eyes.
  - Requirements table from `requests.md`: requirement · where in the result · met, partly or not · evidence. An unmet requirement is a finding.
  - Every "done" gets a probe of the effect, not of the call: the live API, the hash of the sent file, the service's response, a test run after the last code change. Actions in the "After the check" block are not done yet, and that is fine.
  - The main conclusion is in the first sentences; numbers have a period and units; wording is no stronger than the evidence. For a "why" question the cause must explain the size of the effect; otherwise the first sentences say "explained N of M".
- **SKEPTIC** — tries to refute.
  - For every "because of / due to / all / always" — alternatives: incomplete export or a row cap, wrong period, time zone or filter, join duplicates, a tracking change, seasonality or a one-off event, a small sample. For each, a distinguishing test, and it must be run.
  - Denominators, windows, units and coverage of the key numbers.
  - In the fast team — up to five key numbers with its own queries.
- **SCOUT** — looks where the author did not. "Why": break the effect down — the process step by step, outages, inputs, measurement. "Completeness": its own list from the live system and a probe of the data itself, not of the settings.
- **BLIND** — sees `blind.md` and `defs.md` but not the values, and recomputes the key numbers its own way. Cannot — "cannot recompute" with the reason.
- **CODE** — tests in a temporary copy, edge cases, a check of the final file.
- **SOURCE** — primary sources and the live state of systems instead of retellings.
- **SECOND MODEL** — the SKEPTIC role through the CLI of another family. It gets a copy of the run folder without secrets or personal data: mask keys, tokens, passwords, phone numbers, emails and document numbers; leave out row-level exports with personal fields.
- **ARBITER** — a fresh agent that did neither the work nor the check. It gets the disputed findings with both sides' evidence and decides for each: fix or reject, severity, duplicate of what.
- **FIX REVIEW** — the diff and the list of problems. For each: closed or not (an item with no answer is not closed), and what the fixes broke.

## Lite

`/antifake lite` turns the mode on until `/antifake lite off`. Write the flag to `~/.antifake/lite.json` so it survives context compaction.

For every answer with a new result (numbers, causes, "done", "impossible", code, a report), before showing it:
1. Write the answer to a file and run the gate.
2. One checker — header plus this brief: the gate signals; up to five key numbers, the main one with its own query; one effect probe for every "done"; every "impossible" — a primary source or another way; one test of the strongest alternative to the main conclusion; whether the user's question is answered.
3. Fix the findings; the same agent briefly checks the fix.
4. Under the answer — one line: `antifake lite: numbers 3, own queries 2 · actions 1 · fixed 1`.

An answer without a new result (a question, a clarification, a short reply) goes out unchecked.

**Switch to a full run** for that answer on your own, without asking and at most once an hour, if:
- lite found two or more confirmed errors in numbers and conclusions;
- or the answer matters (money, a management decision, an irreversible action, text that leaves the conversation) and the main point could not be checked, the main point is wrong, or the conclusion rests on a model or forecast in which lite found a weak spot.

## Never

- Re-run the check to get a clean result.
- Take external actions that findings call for — only propose them to the user.
- Write "impossible" without checking — write "found no way, checked like this" instead.
