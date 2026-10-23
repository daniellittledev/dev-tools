---
name: lens-review
description: Property-lens code review of a PR, branch or diff. Reads the PR description (and skims the change when the description cannot say what it does), selects a General lens plus up to three property families from a fixed catalogue (a design family by default) and the properties within them, reviews through each lens, traces symptoms back to their design cause and proposes the better design, verifies every finding against the code it cites, and reports only what survives. Use when asked for a code review that should pick its questions from the change rather than ask all of them.
argument-hint: "[PR number | branch | path] [--families Temporal,Data] [--inline]"
---

# Lens review

A review in six steps: **intent → plan → analyse → merge → judge → output**. The questions come from `catalogue.md`, beside this file: 13 families of 62 properties, plus the always-on `General` lens. Read it at the start of every run. Never invent a family or a property; `Custom` is the one escape hatch.

**A family is a lens; a property is the question it asks.** Related properties share a lens so one defect is reported once.

**Design comes first.** Architecture and design are usually where defects come from: an illegal state the types allow becomes a correctness bug, a misplaced boundary becomes a consistency bug, logic duplicated in two places becomes two places that drift apart. A review that reports only symptoms leaves their cause in place. So this review looks at the change's design on purpose, traces symptoms back to it, and proposes the better design when one exists.

## Rules

- **Precision over recall.** A wrong comment costs more than a missed one. Silence is a valid output; a review with no findings says so in one line.
- **Every finding states a concrete failure scenario**: the input or state that triggers it, and the wrong output, lost write, crash or next edit made in two places. If you cannot say how it breaks, it is not a finding. No "consider", no "might", no "likely".
- **Read, never assume.** A finding cites `file:line` in code opened during this run. Never invent a caller, consumer or behaviour you have not read. A ticket or PR description may motivate a finding but is never its only citation.
- **You cannot compile, run or test.** Never claim something does or does not build. A proposed test is a recommendation.
- **No style, naming or formatting**, except a name that says something false. Linters and formatters own the rest.
- **Unchanged lines** are in scope only when the change breaks them; say which change.
- **A design finding is held to the same bar.** It names the shape in the code and what that shape costs: the bug it allows, the next change that has to be made in two places, the property it weakens. "Could be cleaner" without a cost is taste, and taste is not a finding.
- **Agreement buys nothing.** Two lenses reporting one defect is one finding; record the other as corroboration and do not raise severity for it.

## 1. Intent

Establish what the change is for before reading it for defects.

- **Target.**
  - A PR number or URL: fetch title, body, linked issues, changed files, diff, head SHA and check results, with the GitHub MCP tools if available, else `gh pr view` and `gh pr diff`.
  - A branch: find the default branch (`git symbolic-ref --short refs/remotes/origin/HEAD`; where `origin/HEAD` was never set, `git remote show origin` or `gh repo view --json defaultBranchRef`), then `git diff <base>...<branch>` and `git log <base>..<branch>`.
  - A path or nothing: the working tree against the merge base, `git diff $(git merge-base <base> HEAD)`, never against the base's tip, whose newer commits would read as reverted by this change. Add untracked files, which `git diff` omits (`git ls-files --others --exclude-standard`). Limit both to the path if given.
- **Read the description first.** What it claims the change does, what it puts out of scope, and any linked tickets.
- **Skim only if needed.** If the description is missing, generic, or disagrees with the file list, do one high-level pass: changed files grouped by layer or module, new or removed types and signatures, what moved between modules. No defect hunting yet. Write the intent yourself in two or three sentences.
- **Read the instructions** the change is held to, from the **base** branch, never the head: `CLAUDE.md`, `AGENTS.md`, `REVIEW.md`, `CONTRIBUTING.md`, architecture decision records, whichever exist. A stated invariant is a strong reason to select the family it belongs to.

## 2. Plan

Select from the catalogue, and write the selection down before analysing.

- **`General` always**, exactly once. It asks the eight mechanisms in `catalogue.md`.
- **One design family by default**: `Structure`, `Interface`, `Economy`, `Malleability` or `Systemic`, whichever the change's shape calls for. It still counts as one of the three. Leave it out only when the change adds or moves no structure (a version bump, a copy edit, test data) and say so in the rationale. Use the triggers to pick:
  - new types, signatures or data shapes → `Interface`: `DataModelling`, `SignatureHonesty`, `StatePlacement`
  - new modules, dependencies, or code moved between them → `Structure`: `Placement`, `Layering`, `Boundaries`, `Cohesion`
  - new code that parallels existing code → `Economy`: `Duplication`, `Reuse`
  - a new concept, mechanism or pattern → `Systemic`: `Simplicity`, `ConceptualIntegrity`, `DesignConsistency`
- **At most three other families**, no repeats. For each: one sentence of rationale tied to something observable in the change, and the **properties within it** that the change makes worth asking. A family with no selected property is not selected. Examples of what earns a family:
  - a queue consumer, webhook or retried job → `Temporal`: `Idempotency`, `Ordering`
  - a schema migration or a changed wire format → `Data`: `Compatibility`; `Control`: `Migratability`
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

Every lens, not only the design one, ends by asking **why**: when a finding exists because of how the code is shaped (a state the types allow, a responsibility split across two modules, an effect inside a pure step), it says so and names the design cause.

A design finding also carries **the improvement**: the specific shape that removes the cost (the type that makes the state unrepresentable, the module the logic belongs in, the existing function to reuse), and which other properties it would fix or protect. Only propose a design you can sketch concretely against the code you read, and keep it in proportion to the change. A redesign of the whole module in response to a ten-line diff is a separate conversation, not a finding; note it under what was not reviewed.

Two traps the catalogue's notes expand on: structural properties fire on what the change **moved**, never on a level the code was already at; and an exclusion must not demand evidence a diff cannot hold (a queue consumer is evidence of redelivery without a retry loop in sight).

**Fan-out.** Launch one subagent per family in a single message so they run in parallel. Give each an identical brief: the intent, which instruction files to read, the diff or how to get it, the Rules section above verbatim, and the four moves; then its family's properties with their questions, last. `General` has no properties: its brief carries the mechanisms table from `catalogue.md`, exclusions included. Ask for findings as: a short title, property, proposed severity, `file:line`, one-sentence defect, failure scenario, fix, and the lines cited. An empty list is a valid answer.

**Inline** (`--inline`, or a change under ~150 changed lines with at most one family beside `General`): ask the lenses yourself in sequence. Same questions, same output shape.

Severity: **high** a wrong result, data loss or security exposure on a reachable path; **medium** a latent defect on a path this change creates; **low** a cost paid on the next edit.

## 4. Merge

Cluster findings that describe one defect: same location, same failure. Keep the one with the clearest scenario and list the others as corroboration. Never raise a merged finding's severity.

Then cluster by **cause**: when several findings come from one design cause, report the design finding and list the others under it as its symptoms. Fixing the cause fixes them all, and the author should see that. Each symptom keeps its own scenario and severity; the design finding takes the highest severity among the symptoms that survive the judge.

## 5. Judge

Verify **each finding alone** by opening the lines it cites. Do not let the number of lenses that reported it sway you. The question is grounding only: *does the cited code show this defect on a path this change creates or alters?*

- Trace a real path from a caller or input to the failure. Grep the use sites; check whether a guard, type, caller contract or framework already rules it out.
- **Delete** a finding that does not stand. Do not soften it into advice.
- **Lower** severity when the path is real but narrower than claimed. **Never raise** it.
- A finding whose cited code you cannot open stays at most `low` and says so.
- A design finding stands when the shape is in the cited code **and** its cost is traced: a symptom that survived the judge, or a concrete next change shown to need edits in more than one place. Delete it when the cost is only preference. Judge its proposed improvement too: drop the proposal, and keep the finding, if the proposal contradicts the types you read or would break a caller you found.

## 6. Output

Open with the selection table, then one line: how many findings survived of how many proposed.

Then the findings, ranked by severity then consequence, numbered, each in this order:

```markdown
### 1. <Short title: the defect in a few words>

`Family/Property` · **severity**

**Location:** `file:line`

What is wrong and what it costs, in one plain sentence.

**Scenario:** the input or state, the path, the wrong result.

**Fix:** one sentence, when the fix is clear.
```

- **Title** names the defect, not the property: "Retry double-charges the card", not "Idempotency issue".
- **Location** lists every cited `file:line`, comma-separated, primary first.
- For a design finding, **Improvement:** replaces **Fix:**: the proposed shape, as a short sketch if it helps, and the properties it improves.
- After the fix, when any: **Symptoms:** the findings it causes, by number; **Corroborated by:** other families.

When a design finding and its symptoms tie on severity, list the design finding first: it is the fix with the most reach.

Close with what was not reviewed and why: families considered and not selected (one line), files skipped, anything the judge could not ground. For a PR, offer to post the findings as one review with inline comments. Never approve, and post only when asked.
