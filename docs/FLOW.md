# How the check works

The rule is old: the one who did the work does not check it. Before the user sees a result, agents that did not produce it check it, and "checked" counts only with a command and its output. What each checker does is in [ROLES.md](ROLES.md); the skill itself is [`skills/antifake/SKILL.md`](../skills/antifake/SKILL.md).

## The pipeline

```
author does the task
  → freeze: the result, the final text, the user's requests and the author's command outputs are saved and hashed
  → gate: mechanical signals — numbers without a source, "done" without a trace,
          "all / always / fully", "impossible / nowhere / does not exist"
  → checkers in parallel, each a fresh read-only agent with its own question
  → trace check: every "checked" must have a command and its output
  → arbiter: a fresh agent rules on disputed findings only
  → author fixes; fix review checks the diff (a second, last round if something major is left)
  → card: what was recomputed, what was only matched against the author's words, what was not checked
  → after the card: actions promised "after the check" are taken and probed
```

While the check runs, the user does not see the result.

## Modes

| Mode | Who checks | Time |
|---|---|---|
| **lite** | one checker on every answer with a new result, switched on for a session | a few minutes |
| **fast** (default) | ACCEPTANCE and SKEPTIC | about 15 minutes |
| **full** | fast team plus BLIND, CODE or SOURCE, depending on the task | longer |
| **hard** | full team plus a model from another family; the arbiter rules on every finding | the longest |

**Which team.** Fast by default. The agent picks full on its own, without asking, when money or a contract depends on the result or the numbers go to management or outside. By task type, full adds:
- analytics — BLIND;
- report — BLIND and CODE;
- code — CODE;
- infrastructure — SOURCE and CODE;
- anything else — SOURCE.

**SCOUT** joins any team when:
- the user asks "why / what happened / where did it come from";
- the task is about infrastructure or access;
- the text says "all / nowhere / none / fully / impossible / does not exist".

**The second model** joins hard mode.

## Findings and fixes

- A finding has a location, the problem, its impact and evidence — a command with its verbatim output, or a quote from a primary source — plus a one-sentence fix. A suspicion that was not tested is not a finding.
- **Major** means it changes a conclusion, a key number or a decision, or breaks the user's requirement. Everything else is minor.
- A finding whose evidence is in the checker's own log goes straight to fixes. Untraced findings, recount mismatches and the second model's findings go to the ARBITER. The author may dispute a finding only with new evidence.
- **One problem is one item.** Findings from different checkers about the same thing are merged; the item weighs as its heaviest finding.
- The author fixes the result and the final text, leaving no traces of the check in them. A claim that cannot be proven is rewritten as a hypothesis. Declining to fix a major problem gives NOT ACCEPTED.
- FIX REVIEW checks the diff. If something major is left, or a fix broke something major, there is a second and last round. Minor leftovers become caveats.

## The card

- **ACCEPTED** — no findings.
- **ACCEPTED WITH FIXES** — there were findings, all closed.
- **NOT ACCEPTED** — a major problem was left unfixed.
- **NOT FULLY VERIFIED**:
  - a checker failed or broke the rules;
  - a major item is still open;
  - a fix broke something major;
  - a key number has neither a source nor a recount;
  - a gate signal was left unanswered;
  - in hard mode, the second model did not answer.

An agent's refusal, a limit or a failure is never "accepted". The user sees exactly the card and the final text, nothing on top.

## Lite mode

Switched on for a session. Every answer with a new result — numbers, causes, "done", "impossible", code, a report — goes through one quick checker before it is shown. The checker:
- answers the gate signals;
- gets up to five key numbers, the main one by its own query;
- probes the effect of every "done";
- checks every "impossible" another way;
- tests the strongest alternative to the main conclusion;
- checks that the user's question is answered.

Answers with nothing new go out unchecked.

**Lite escalates to a full check of that answer on its own,** without asking and at most once an hour, if:
- it found two or more confirmed errors in numbers and conclusions;
- or the answer matters (money, a management decision, an irreversible action, text leaving the conversation) and the main claim could not be checked, is wrong in a sloppy draft, or rests on a model or forecast with a weak spot.

Importance alone, one fixable error or a false "done" are ordinary lite findings.

## The second model

A model from another family, called through its CLI with the user's subscription — no API key. On the first run the skill:
1. finds the CLI (`codex` for Claude, `claude` for GPT);
2. checks the sign-in;
3. makes a probe call.

The model reads a package with keys, passwords, contacts and personal fields masked, in a read-only sandbox. A Claude checker then runs the checks the model proposes; the model's own remarks never count as evidence.

## Your own traps

No examples ship with the skill: on the first run the agent looks at what the user works on and writes `~/.antifake/traps.md` — the typical ways a result in this work goes wrong, each with a one-line check. The checkers read it with their brief. After every miss the user reports, a trap is added, so the skill adapts to the user's work.

## Language

Everything the user sees is in the user's language:
- commands are understood in any language;
- checkers answer in the language of the user's messages;
- the card is translated without changing numbers or statuses.
