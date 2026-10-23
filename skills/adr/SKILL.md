---
name: adr
description: Record an architecture decision as a short, lasting decision record. Captures the context that forced it, the decision stated as a rule later changes can be checked against, the alternatives turned down with their reasons, and the consequences accepted. Finds the repository's existing records and follows their format, or starts a directory. Drafts from a PR, a branch or the conversation, and supersedes an earlier record when a decision changes. Use when asked to write, record or document a decision or an ADR, or when a change makes a decision that future changes must respect.
argument-hint: "[title | PR number | branch] [--supersedes <number>] [--status proposed|accepted]"
---

# Decision record

A decision record answers the question a newcomer asks a year later: why is it like this, and can I change it? The code shows what was built. A PR description explains one change and is then buried. A record keeps the decision, the forces behind it and the alternatives turned down in the repository, beside the code, for as long as the decision holds. Reviews can then hold later changes to it.

## Rules

- **One decision per record.** If the source holds several, write one record each, and only for decisions a later change could get wrong.
- **The decision is a rule.** State it so that a later change can be checked against it: "Domain types never reference persistence types; repositories map at the edge", not "We discussed layering".
- **Why, from the forces.** The context names what made a decision necessary and what constrained it: a requirement, a failure, a limit, a deadline. Without the forces, a reader can't tell whether the decision still applies.
- **Alternatives, fairly.** Give each rejected option at its strongest, and the specific reason it lost. A straw man tells the reader nothing about when to revisit.
- **Consequences, including costs.** What gets easier, what gets harder, what is now ruled out, and the signal that would mean it is time to revisit.
- **Short and plain.** One screen. Short sentences, everyday words, no filler, no hype.
- **True to the source.** Record only what the PR, the code or the user supports. A reason that cannot be found is asked for, not invented.
- **Accepted records do not change.** A changed decision gets a new record that supersedes the old one. The old record's status line is the only part of it you edit.

## 1. Find the home

- **Existing records.** Look for `docs/adr/`, `doc/adr/`, `docs/decisions/`, `adr/`, `architecture/decisions/`, or a `.adr-dir` file. Read the two most recent records for their numbering, filename pattern, headings and status words. Follow them exactly, even where they differ from the format below.
- **None yet.** Use `docs/adr/`, numbered `0001-<kebab-title>.md`, in the format below. Say in the report that this starts the convention.
- **An index.** If the directory has a `README.md` or index listing the records, it gets a line for the new one.

## 2. Gather

- **From a PR or branch.** Its description (a "Key decisions" section is the place to start), linked issues, commit messages, review discussion and the diff. Read the code that carries the decision: the record cites where it lives.
- **From the conversation.** What the user said, and the code it refers to.
- **The forces.** If the context or the alternatives cannot be found, ask the user one direct question for each gap ("What made the queue necessary now?", "What else was considered?"). Then stop and wait.
- **With `--supersedes`**, read the old record. The new one says what changed in the forces to overturn it.

## 3. Write

```markdown
# <Number>. <The decision, as a short statement>

Status: <Accepted | Proposed> · <date>
Supersedes: [<number>. <title>](<file>)        <- only when it does

## Context
The forces: what made a decision necessary and what constrained it.
Two to five sentences.

## Decision
We will <the rule>. Where it applies, and where it does not.
Where in the code it lives, if one place carries it.

## Alternatives
- **<Option>.** What it offered, and why it lost.

## Consequences
What gets easier, what gets harder, what is ruled out.
When to revisit: the signal that would mean the forces have changed.
```

The status is `Accepted` unless `--status` says otherwise. The date is today, or the merge date when recording a decision from a merged PR.

## 4. Check

Read the draft as a newcomer would, cold:

- Could they tell from the Decision alone whether their change breaks it?
- Does each alternative carry a reason specific enough to know when it would win?
- Is every claim backed by the source?

## 5. Deliver

- Write the record, and the index line if there is an index.
- **Superseding**: set the old record's status to `Superseded by [<number>. <title>](<file>)`. Change nothing else in it.
- **When the source is an open PR**, offer to commit the record to that PR's branch: the decision and the code that makes it change together. Otherwise, commit only when asked.
- **Report** the file's path and its one-line decision.
