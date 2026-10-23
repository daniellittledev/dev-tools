# dev-tools

Skills for development with Claude Code, published as one plugin. This repository has no code to build or test: the skills are the product, and they are prose.

## Layout

- `skills/<name>/SKILL.md`: one directory per skill. The directory name is the skill's name.
- Supporting files sit beside `SKILL.md` and are referenced as "beside this file" (for example `skills/lens-review/catalogue.md`).
- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`: the plugin manifest, and the repository acting as its own single-plugin marketplace.
- `README.md`: one table row per skill, in alphabetical order, plus install instructions.

## Writing a skill

Follow the shape the existing skills share. Read `skills/pr/SKILL.md` before writing a new one.

**Frontmatter.**
- `name` matches the directory.
- `description` says what the skill does in plain terms, then ends with "Use when ..." and the requests that should trigger it. The description decides whether the skill is picked, so name the trigger words a user would actually say.
- `argument-hint` lists the arguments and flags.

**Body, in this order.**
1. A `#` title and one opening paragraph: the problem the skill solves and the principle it works from. This is the why every rule below follows from.
2. `## Rules`: bullets, each opening with a **bold lead** that states the rule, followed by its reason or its edge. The rules hold across every step.
3. Numbered steps, `## 1. <Verb>`, in the order they run. Gather comes first. Checking your own output comes before publishing it.
4. The last step says what to report, and how briefly.

**Style.**
- Plain, short sentences, everyday words, active voice, the imperative mood.
- Lead with the point. No filler, no hype ("robust", "comprehensive"), no hedging ("consider", "might").
- Make rules concrete. Give a quoted example of the right and wrong output where the difference is subtle. Name the wrong move as well as the right one, with its reason.
- Bold for the lead phrase of a rule or step, not for emphasis inside a sentence.

**Behaviour every skill keeps.**
- **Read, never assume.** Claims cite what was opened during the run.
- **True to what was seen.** Never claim a test, check or measurement the run did not see.
- **Ask when the why is missing.** One direct question beats a guessed reason.
- **Outward actions only when asked.** Never approve or merge. Post, publish or push only where the skill's purpose is to do so, or the user asks.
- **Tools.** Prefer the GitHub MCP tools when available, with `gh` as the fallback. Name no model or tool brand unless the skill is about it.

## Self-contained skills

A skill must work when its directory is copied alone into `~/.claude/skills/`, as the README's install instructions allow. So a skill never depends on another skill being installed, and never refers to another skill by name as a step to run. Small duplication across skills (for example, how to find the default branch) is deliberate: it is the price of independence.

## Adding or changing a skill

- Add or update the skill's row in `README.md`.
- Keep the manifests' `description` and `keywords` true to the set of skills the plugin ships.
- In the commit and the PR, explain why the skill exists and which gap it fills beside the others.
