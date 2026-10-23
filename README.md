# dev-tools

Tools for development with Claude Code.

## Skills

| Skill | What it does |
|---|---|
| [`lens-review`](skills/lens-review/SKILL.md) | Property-lens code review. Reads the PR's intent, picks a General lens plus up to three families from a fixed [catalogue](skills/lens-review/catalogue.md) of 62 properties, reviews through each with design first (tracing symptoms back to their design cause and proposing the improvement), verifies every finding against the code it cites, and reports only what survives. |
| [`pr`](skills/pr/SKILL.md) | Opens a pull request, or rewrites an existing description, for a reviewer reading cold: plain high-level language, the why behind the change, and the key design decisions with their reasons and the alternatives turned down. |

### Install

As a plugin, which installs every skill and keeps them updated:

```
/plugin install dev-tools --marketplace daniellittledev/dev-tools
```

Then run `/dev-tools:lens-review [PR number | branch | path]` or `/dev-tools:pr [base branch]`.

Or copy a single skill's directory into your personal or project skills directory:

```bash
mkdir -p ~/.claude/skills && cp -r skills/<skill> ~/.claude/skills/              # every project
mkdir -p <repo>/.claude/skills && cp -r skills/<skill> <repo>/.claude/skills/    # one project
```

Then run `/lens-review [PR number | branch | path]` or `/pr [base branch]`.

## Templates

| Template | What it is |
|---|---|
| [`AGENTS.md`](templates/AGENTS.md) | General user level preferences for coding agents: brevity, functional style, self-documenting code, and commit and PR conventions. |

### Install

Claude Code reads user level instructions from `~/.claude/CLAUDE.md`, Codex from `~/.codex/AGENTS.md`:

```bash
mkdir -p ~/.claude && cp templates/AGENTS.md ~/.claude/CLAUDE.md
mkdir -p ~/.codex && cp templates/AGENTS.md ~/.codex/AGENTS.md
```
