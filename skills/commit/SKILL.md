---
name: commit
description: Commit changes with a message that explains why. Writes a short imperative subject and a body giving the reason for the change and any decision a future reader of `git log` or `git blame` would question, follows the repository's message convention, and can split unrelated changes into separate commits. Use when asked to commit, write a commit message, or split changes into commits.
argument-hint: "[--all] [--split] [--amend]"
---

# Commit

A commit message is read by someone who has found this line in `git blame` and wants to know why it is like this. The diff already says what changed. The message says why it changed, and why this way when another way was plausible. It also feeds the pull request: a branch whose commits explain themselves already holds most of its own description.

## Rules

- **Explain why.** The subject says what the change achieves. The body says why it was needed and why it was done this way. A body that only restates the diff ("Update parse to handle null") is cut or rewritten as its reason ("Webhooks from forks arrive without a sender, and parsing threw on them").
- **One logical change per commit.** A refactor and the feature built on it are two commits. A rename and a bug fix are two commits. Each commit should be one a reviewer could revert on its own.
- **Plain and short.** Short sentences, everyday words, active voice. No filler, no hype, no hedging. When the subject already makes the why obvious ("Fix typo in install instructions"), the subject alone is enough.
- **True to the diff.** Describe only what is staged. Never claim a test or check you did not see run.
- **The repository's convention.** If its instructions or recent history use a format (Conventional Commits, a ticket prefix, required trailers), follow it.
- **Never** commit secrets or credentials, skip hooks (`--no-verify`), or rewrite a commit that has been pushed.

## 1. Gather

- **What will be committed.** `git status` and `git diff --cached`. If nothing is staged:
  - with `--all`, stage every tracked change, plus untracked files once you have listed them;
  - otherwise, show what changed and ask what belongs in this commit.
- **Check the staged files.** Look for anything that should not be committed: `.env` files, keys, tokens, build output, large binaries, editor files. Stop and ask about any you find.
- **The convention.** `git log -n 20 --format=%B` and any instruction files (`CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`) for the subject format, body style and required trailers.
- **The reason.** In order of trust: what the user said in this conversation, the linked issue (from the branch name or the conversation), comments in the diff. If the why is not obvious from the diff and cannot be found, ask one direct question ("What problem does this solve?"). A guessed reason is worse than none.

## 2. Split

With `--split`, or when the staged changes hold more than one logical change:

- Group the changes by purpose, and order the groups so each commit builds and makes sense alone: refactors before the behaviour that needs them.
- Show the plan (each commit's subject and its files) and wait for the user to confirm it.
- Stage each group by file. When one file holds changes for two groups, stage its hunks with `git apply --cached` on a patch of just those hunks. If that is not clean, ask whether to commit the file whole.

Without `--split`, when the staged changes clearly hold unrelated work, say so once and offer to split. Then do what the user chooses.

## 3. Write

```
<Subject: imperative, under about 60 characters, what the change achieves>

<Why: the problem or need, in one to three sentences.>

<A decision a reader would question, if any: what was chosen, why,
and the alternative turned down.>

<Trailers: Fixes #123, and any the repository or user requires.>
```

Wrap the body at 72 characters. Leave out any part with nothing true to say. Use the repository's format in place of this one where it has one.

## 4. Commit

- Commit with the message passed in a file or heredoc, so the formatting survives.
- **If a hook fails**, the commit did not happen. Fix what the hook reports, stage the fix, and commit again as a new commit. Do not amend: there is nothing to amend.
- **With `--amend`**, rewrite the last commit's message or contents only if it has not been pushed (`git branch -r --contains HEAD` is empty). Otherwise stop and say why.
- Do not push unless asked.

**Report** the commit's short hash and subject, one line per commit.
