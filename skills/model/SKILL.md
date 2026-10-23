---
name: model
description: Design a feature types-first, before writing its implementation. Turns a feature request into a domain model where illegal states are unrepresentable, workflows are pure functions with honest signatures, effects sit at the edges, and the whole feature is one vertical slice in the codebase's own idiom. Use when asked to model, design or plan a feature, a domain or a data shape before coding it, or when a change needs new domain types.
argument-hint: "[feature description | issue ref] [--write]"
---

# Model

Most defects that reach review were designed in. A status field paired with optional fields lets a cancelled order keep a delivery date. A string can hold an email that was never checked. A function that hides its database call can't be tested without one. A model built types-first rules these out before any code depends on them. That is cheaper than finding them in review and refactoring after.

This skill produces the model: the rules, the types that enforce them, the workflow signatures and where the slice lives. It does not write the implementation.

## Rules

- **Types before code.** Produce data definitions and function signatures, not function bodies.
- **Make illegal states unrepresentable.** When a value is one of several alternatives, use a sum type, not flags or nullable fields. When an entity moves through states, give each state its own type carrying only the fields valid in it (`UnverifiedEmail` and `VerifiedEmail`, not `Email` with an `isVerified` flag). When a primitive has a rule, use a constrained type with one way to construct it (`Quantity` from 1 to 100, not `int`).
- **Honest signatures.** The type says what the function does. A function that can fail returns its failure in the type, as a closed set of domain errors. A function that does I/O shows it in its signature or takes it as a parameter.
- **Pure core, effects at the edges.** A workflow's decisions are pure functions from input and current state to a result. Loading, saving, sending and the clock are at the edges, passed in.
- **One vertical slice.** Everything the feature needs lives together, in one place, from the boundary type to the persistence call. Code goes in a shared module only when a second slice already uses it.
- **Every type earns its place.** Each type enforces a rule you can name. No speculative generality, no layer for its own sake. The smallest model that holds the rules is the right one.
- **The codebase's idiom.** Use the language's own encoding: discriminated unions in TypeScript, sealed types in Kotlin, Java or C#, enums with data in Rust or Swift, union types in F#, Haskell or OCaml. Follow the conventions the code already uses for errors, results and modules. Don't add a library to model with.
- **Read, never assume.** Reuse an existing type only after opening it. Cite what you build on.

## 1. Understand

- **The request.** The argument, the linked issue, and what the user said in this conversation.
- **The codebase.** The existing domain types the feature touches, one or two neighbouring slices to learn the structure from, and the instructions: `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, architecture decision records.
- **The rules.** Write the domain's rules as a numbered list in plain language: what must always be true, what may happen in which state, and what can go wrong. These are the specification. Each type and signature that follows traces back to one.
- **The gaps.** A rule that would change the model's shape and cannot be found (can an order be part-shipped? can a user have two emails?) is a question for the user. Ask up to three such questions at once, then stop and wait. A guessed rule becomes a wrong type that every caller depends on.

## 2. States and transitions

- **The nouns.** The entities and values the rules talk about.
- **Their lifecycles.** For each entity that changes: its states, what is known in each, and the transitions between them. A field that is "set later" is a sign of a state you haven't named yet.
- **The workflows.** Each command a user or system issues, the state it starts from, the state or event it produces, and the ways it can fail as domain outcomes ("out of stock"), not exceptions.

## 3. Types

- **Constrained values** for every primitive with a rule, each with one constructor that validates and returns a result.
- **A sum type** for every set of alternatives and every lifecycle. Each case carries only the data valid in it.
- **Records** that compose those, with no optional field unless absence is a real domain meaning.
- **An error type** per workflow: the closed set of ways it can fail.
- **Boundary types** stay separate. Unvalidated input and wire or database shapes are their own types, mapped to domain types at the edge. The domain never sees them.

For each rule, record how it is enforced: **by type** (cannot be constructed), **by constructor** (checked once at creation), **by workflow** (checked in a decision), or **not enforced**, with the reason. Prefer the first. Every "not enforced" must be deliberate.

## 4. Workflows

For each workflow, a signature in three stages:

1. **Validate**: unvalidated input → domain types, or validation errors.
2. **Decide**: domain types and current state → new state or events, or domain errors. Pure.
3. **Act**: persist and publish the decision. The effects, at the edge, with their dependencies as parameters.

Write them as function types. Name the dependencies (`LoadOrder`, `SaveOrder`, `Now`) as function types too, not as service interfaces with many methods.

## 5. Slice

Where each part goes, following the structure of the neighbouring slices you read: the folder or module, which files, and the one place the slice is wired in. List anything it must change outside itself (a shared type, a route table, a migration) and why.

## 6. Check

Before presenting, attack the model:

- **Construct an illegal state.** For each rule, try to build a value that breaks it using only the public constructors. If you can, fix the type or move the rule to "not enforced" with a reason.
- **Walk the scenarios.** Run the main path and each failure through the workflow signatures. Every step needs its data in hand, from its inputs or a declared dependency.
- **Cut.** Remove any type that enforces no rule and any stage that only forwards its input.

## 7. Output

In the repository's language, in this order:

1. **Rules**: the numbered list, each with how it is enforced.
2. **Types**: the definitions, grouped by entity, with a one-line comment only where a rule is not obvious from the type.
3. **Workflows**: the signatures and their dependencies.
4. **Slice**: the layout.
5. **Open**: questions still unanswered, and rules not enforced with the reason.

With `--write`, also add the types and the workflow function types to the codebase where the slice says, with no implementation bodies. Run the repository's typecheck or compile step and report what you saw. Never write implementation unless asked.
