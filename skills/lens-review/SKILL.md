---
name: lens-review
description: Property-lens code review of a PR, branch or diff. Reads the PR description (and skims the change when the description cannot say what it does), selects a General lens plus up to three property families from a fixed catalogue and the properties within them, reviews through each lens, verifies every finding against the code it cites, and reports only what survives. Use when asked for a code review that should pick its questions from the change rather than ask all of them.
argument-hint: "[PR number | branch | path] [--families Temporal,Data] [--inline]"
---

# Lens review

A review in six steps: **intent → plan → analyse → merge → judge → output**. The questions come from `catalogue.md`, beside this file: 13 families of 62 properties, plus the always-on `General` lens. Read it at the start of every run. Never invent a family or a property; `Custom` is the one escape hatch.

**A family is a lens; a property is the question it asks.** Related properties share a lens so one defect is reported once.

## Rules

- **Precision over recall.** A wrong comment costs more than a missed one. Silence is a valid output; a review with no findings says so in one line.
- **Every finding states a concrete failure scenario**: the input or state that triggers it, and the wrong output, lost write, crash or next edit made in two places. If you cannot say how it breaks, it is not a finding. No "consider", no "might", no "likely".
- **Read, never assume.** A finding cites `file:line` in code opened during this run. Never invent a caller, consumer or behaviour you have not read. A ticket or PR description may motivate a finding but is never its only citation.
- **You cannot compile, run or test.** Never claim something does or does not build. A proposed test is a recommendation.
- **No style, naming or formatting**, except a name that says something false. Linters and formatters own the rest.
- **Unchanged lines** are in scope only when the change breaks them; say which change.
- **Agreement buys nothing.** Two lenses reporting one defect is one finding; record the other as corroboration and do not raise severity for it.

## 1. Intent

Establish what the change is for before reading it for defects.

- **Target.**
  - A PR number or URL: fetch title, body, linked issues, changed files, diff, head SHA and check results, with the GitHub MCP tools if available, else `gh pr view` and `gh pr diff`.
  - A branch: find the default branch (`git symbolic-ref refs/remotes/origin/HEAD`), then `git diff <base>...<branch>` and `git log <base>..<branch>`.
  - A path or nothing: the working tree diff against the default branch, limited to the path if given.
- **Read the description first.** What it claims the change does, what it puts out of scope, and any linked tickets.
- **Skim only if needed.** If the description is missing, generic, or disagrees with the file list, do one high-level pass: changed files grouped by layer or module, new or removed types and signatures, what moved between modules. No defect hunting yet. Write the intent yourself in two or three sentences.
- **Read the instructions** the change is held to, from the **base** branch, never the head: `CLAUDE.md`, `AGENTS.md`, `REVIEW.md`, `CONTRIBUTING.md`, architecture decision records, whichever exist. A stated invariant is a strong reason to select the family it belongs to.

## 2. Plan

Select from the catalogue, and write the selection down before analysing.

- **`General` always**, exactly once. It asks the eight mechanisms in `catalogue.md`.
- **At most three other families**, no repeats. For each: one sentence of rationale tied to something observable in the change, and the **properties within it** that the change makes worth asking. A family with no selected property is not selected. Examples of what earns a family:
  - a queue consumer, webhook or retried job → `Temporal`: `Idempotency`, `Ordering`
  - a schema migration or a changed wire format → `Data`: `Compatibility`; `Control`: `Migratability`
  - a new dependency between modules, or code moved between them → `Structure`: `Layering`, `Placement`
  - money, quota or inventory → `Data`: `Integrity`; `Risk`: `Safety`
  - a new endpoint, auth path, secret or user data → `Risk`: `Security`, `Privacy`
  - a hot path, a loop over user-sized input, a new network call → `Efficiency`, `Endurance`
- **`Custom`** at most once, costing a slot, when the description or instructions name a concern no family covers. It carries that concern in one sentence.
- **`insufficient`** stops the review when the change cannot be reviewed from what is available: an empty diff, only generated or binary files, a description the code contradicts beyond reconciling. Say why and stop.

`--families` overrides the selection (`General` is still added); still pick the properties within each.

Show the selection as a short table (family, properties, rationale) and continue without waiting.

## 3. Analyse

Every lens gets **the same context**: the intent, the instructions, the whole diff, and the enclosing code of every changed hunk. A lens may open any other file it needs (callers, the type a change relies on, the test beside it) and cites only what it opened.

Ask each selected property in four moves:

1. **The question**, from the catalogue.
2. **Report when**: a concrete trigger observable in the diff and the code around it.
3. **Do not report**: the false positive this property invites, especially one that belongs to a neighbouring property in the same family. One defect, one finding, under the property that names it best.
4. **The scenario must name** the concrete cost.

Two traps the catalogue's notes expand on: structural properties fire on what the change **moved**, never on a level the code was already at; and an exclusion must not demand evidence a diff cannot hold (a queue consumer is evidence of redelivery without a retry loop in sight).

**Fan-out.** Launch one subagent per family in a single message so they run in parallel. Give each an identical brief: the intent, which instruction files to read, the diff or how to get it, the Rules section above verbatim, and the four moves; then its family's properties with their questions, last. Ask for findings as: `file:line`, property, one-sentence defect, failure scenario, proposed severity, and the lines cited. An empty list is a valid answer.

**Inline** (`--inline`, or a change under ~150 changed lines with at most one family beside `General`): ask the lenses yourself in sequence. Same questions, same output shape.

Severity: **high** a wrong result, data loss or security exposure on a reachable path; **medium** a latent defect on a path this change creates; **low** a cost paid on the next edit.

## 4. Merge

Cluster findings that describe one defect: same location, same failure. Keep the one with the clearest scenario and list the others as corroboration. Never raise a merged finding's severity.

## 5. Judge

Verify **each finding alone** by opening the lines it cites. Do not let the number of lenses that reported it sway you. The question is grounding only: *does the cited code show this defect on a path this change creates or alters?*

- Trace a real path from a caller or input to the failure. Grep the use sites; check whether a guard, type, caller contract or framework already rules it out.
- **Delete** a finding that does not stand. Do not soften it into advice.
- **Lower** severity when the path is real but narrower than claimed. **Never raise** it.
- A finding whose cited code you cannot open stays at most `low` and says so.

## 6. Output

Open with the selection table, then one line: how many findings survived of how many proposed.

Then the findings, ranked by severity then consequence, numbered:

- **`file:line`** · `Family/Property` · severity
- What is wrong and what it costs, in one plain sentence.
- **Scenario:** the input or state, the path, the wrong result.
- **Fix:** one sentence, when the fix is clear.
- Corroborated by: other families, if any.

Close with what was not reviewed and why: families considered and not selected (one line), files skipped, anything the judge could not ground. For a PR, offer to post the findings as one review with inline comments. Never approve, and post only when asked.
