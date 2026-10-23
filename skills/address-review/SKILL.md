---
name: address-review
description: Work through the review comments on a pull request. Verifies each comment against the code it cites, fixes and pushes the ones that hold, answers the ones that do not with the path traced, proposes rather than builds the large asks, and leaves every thread with a visible outcome. Use when asked to address, respond to, action or work through review feedback on a PR.
argument-hint: "[PR number | URL] [--reviewer <login>] [--no-push]"
---

# Address review

A review comment is a claim about the code, not an order. Some are right, some are wrong, and some are right about something that does not matter. This skill checks each comment against the code, fixes what holds, answers what does not with a reason the reviewer can check, and leaves no thread silent.

## Rules

- **Verify before acting.** A comment holds when you can trace a real path from a caller or input to the failure it describes, in code opened during this run. Grep the use sites; check whether a guard, type, caller contract or framework already rules it out. A comment you cannot ground gets a reply, not a change.
- **Every thread gets an outcome**: fixed (with the commit), declined (with the reason), proposed (with the shape), or a question back. Never leave one silent. Never resolve one you have not answered.
- **Fix the cause, once.** When several comments share one cause (a state the types allow, logic in two places, an effect inside a pure step), fix the cause and say so on each thread.
- **Stay in scope.** Small, local asks are fixed: a guard, a rename, a test, a one-function refactor. Large asks get a reply with a concrete proposal, and the author decides: multi-file refactors, API or schema changes, open-ended design feedback. If you cannot tell whether an ask is small, treat it as large.
- **In proportion.** When the path is real but far-fetched, or the fix adds more code than the failure is worth, say so with the path you traced instead of building it. Code is a liability. A security comment is never declined on proportion alone: put it to the user.
- **Cheap disagreements are not worth having.** Accept a reviewer's naming or style preference unless it says something false or contradicts the repository's instructions.
- **True to what you did.** A reply names the commit and what it changed. Never claim a check passed that you did not see pass.
- **Never** approve, merge, dismiss a review, or rewrite history that has been pushed.

## 1. Gather

- **The PR.** The argument, else the open PR for the current branch. Fetch its title, body, base, head SHA, diff, check results and every review thread: its comments, authors, resolved state and the `file:line` it anchors to. Use the GitHub MCP tools if available, else `gh pr view` and `gh api graphql` (REST does not expose whether a thread is resolved).
- **Which threads.** Unresolved threads whose last comment is not yours, plus review-body comments that ask for a change. `--reviewer` limits this to one reviewer. A thread whose last comment is yours is waiting on the reviewer; skip it.
- **The branch.** Check out the PR's head branch and bring it level with the remote. If the working tree has uncommitted changes, stop and ask.
- **The instructions** the change is held to, from the base branch: `CLAUDE.md`, `AGENTS.md`, `REVIEW.md`, `CONTRIBUTING.md`, whichever exist. A comment that cites a rule is checked against the rule's text, not the comment's summary of it.
- **The intent.** The PR description. A comment asking for something the description puts out of scope is a scope question, not a defect.

## 2. Triage

For each thread, open the cited lines and their enclosing code, then pick one verdict:

| Verdict | When | Action |
|---|---|---|
| **Fix** | It holds, and the fix is small and in scope | Change the code |
| **Propose** | It holds, and the fix is large or a design choice | Reply with a concrete proposal; leave the thread open |
| **Decline** | It does not hold, or the fix costs more than it prevents | Reply with what rules it out, citing `file:line` |
| **Done** | An earlier commit already fixed it | Reply with that commit |
| **Ask** | You cannot tell what the reviewer wants | Reply with one direct question |

Then cluster by **cause**: when several threads come from one design cause, fix the cause once and give each thread the same commit.

Show the triage as a short table (thread, verdict, one-line reason) and continue without waiting. Proposals and questions are replies; no code waits on them.

## 3. Fix

- **Minimal.** Change what the comment needs, no more. A fix that grows past small becomes a Propose.
- **One commit per cause**, its message saying why, with the thread it answers.
- **Check it.** Run the repository's own fast checks on what you changed: lint, format, typecheck, and the tests near the change. Find them in the instructions, the package scripts, the `Makefile` or the CI config. If a check fails and the fix is not obvious, revert that change and turn its verdict into a Propose that names the failure.
- **Re-read the diff** as the reviewer would. Would this fix draw a new comment? Fix that first.

## 4. Reply

- **Push first**, so every commit a reply names exists on the remote. With `--no-push`, commit locally, draft the replies, show them, and post nothing.
- **Reply on each thread**, in one to three plain sentences:
  - Fix: "Fixed in `abc1234`: <what changed>." Add why when the change is not the obvious one.
  - Decline: what rules the failure out (the guard, the type, the caller contract) with `file:line`, or the path you traced and why the fix is not worth its code.
  - Propose: agree on the problem, then the shape you would build, what it touches, and that it needs the author's call.
  - Done: "Already fixed in `abc1234`."
  - Ask: the one question.
- **Resolve** the Fix and Done threads. Leave Decline, Propose and Ask open: the reviewer closes them.
- **Re-request review** from each reviewer whose changes-requested review you addressed.

## 5. Report

A table of each thread and its outcome, the commits pushed, the checks you ran and their results, and what is now waiting on whom: proposals on the author, questions and declines on the reviewer.
