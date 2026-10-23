# User Preferences

## Writing

- When writing or editing CLAUDE.md or AGENTS.md files, keep them brief and concise.
- NEVER wrap lines in commit messages or PR descriptions.
- Write commit messages and PR descriptions in high level language, include context on why the change was made.
- Connect clauses directly or with commas, don't use em dashes.

## Design

- Prefer functional programming style (pure functions, immutability, composition over mutation).
- Push side effects to the edges, keep the core pure.
- Model the domain with types: make illegal states unrepresentable, parse at boundaries rather than validating everywhere.
- Return errors as values (Result or union types) for expected failures, reserve exceptions for the truly exceptional.
- Organise code by feature (vertical slices), things that change together stay together.
- Treat code as a liability: no speculative abstractions or unused flexibility, but don't avoid a better design or a needed refactor just to keep the diff small.
- Prefer self-documenting code: small functions that break up the logical parts, minimal comments that explain why not what.
- Add structured logging or tracing at boundaries and failure points, never swallow errors silently.
- Test behaviour through public interfaces, not implementation details.

## Working

- Prefer PowerShell (`pwsh`) for scripts.
- For large tasks, write a script to accomplish the task rather than performing steps manually.
- Run the build, tests and linter before calling a task done, and report failures honestly.
- Follow the existing conventions of a codebase when they conflict with these preferences.
- Ask before adding dependencies, changing public APIs, or reformatting unrelated code.
- Ask when a requirement is ambiguous, otherwise act without seeking confirmation on routine steps.
- Suggest an alternative if there is a better way.
