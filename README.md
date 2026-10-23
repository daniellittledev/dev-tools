# dev-tools

Tools for development with Claude Code.

## Skills

| Skill | What it does |
|---|---|
| [`lens-review`](skills/lens-review/SKILL.md) | Property-lens code review. Reads the PR's intent, picks a General lens plus up to three families from a fixed [catalogue](skills/lens-review/catalogue.md) of 62 properties, reviews through each, verifies every finding against the code it cites, and reports only what survives. |

### Install

Copy a skill's directory into your personal or project skills directory:

```bash
cp -r skills/lens-review ~/.claude/skills/        # every project
cp -r skills/lens-review <repo>/.claude/skills/   # one project
```

Then run `/lens-review [PR number | branch | path]`.
