# The catalogue

The fixed set the plan selects from: `General`, then 62 properties in 13 families, in six groups. A property is selectable only inside its family.

## General — always selected

Baseline recall. It carries no properties; it asks how code breaks, by eight mechanisms, each with the false positive it invites.

| Mechanism | Report | Do not report |
|---|---|---|
| **Absence** | A value the code shows can be null, None, undefined or missing is dereferenced, indexed or passed to a callee that requires it, on a path this change creates or alters. First ask what the language rules out: F#, Rust, Swift, Kotlin and Scala make absence a type; C# depends on nullable reference types, TypeScript on `strictNullChecks` (look for the setting); Java, Go, JavaScript, Python and Ruby leave it to the code. | A nullable that a guard, type, assertion or visible caller contract already rules out. |
| **Failure** | An error caught and dropped, mapped to success, logged and continued past where continuing is wrong, or a failure path the caller depends on made unreachable. | A deliberately tolerated failure the code or instructions explain; missing handling for a failure that cannot occur here. |
| **Resources** | A handle, connection, lock, subscription, temp file or budget acquired on a path that can exit without releasing it: early return, exception between acquire and release, acquire per iteration. | A resource owned by a caller, framework or visible using-style construct. |
| **Ordering** | Shared state read or written without the ordering its invariant needs: check-then-act on a value another task can change, mutating a collection iterated elsewhere, an await inside a critical section, a callback on an unexpected thread. | A race on state confined to one task; concurrency speculated because it could happen. |
| **Edges** | An input at the edge of its domain gives the wrong result: empty, zero, negative, the largest value, exactly the limit, one past the end. Name the input. | A boundary visibly handled; a theoretical overflow on a value the domain bounds far below it. |
| **Control flow** | A branch, loop or early return does not do what the surrounding code assumes: inverted condition, unreachable case, off-by-one exit, fall-through, a return that skips work the next line depends on. | A choice the description or comments explain as intended. |
| **Trust** | Untrusted input reaches a sink without the check it needs (query, shell, path, deserialiser, HTML, authorisation), or a secret can reach a log or response. Name the source and the sink. | A sink reached only by constants or already validated values. |
| **Contracts** | The change alters the shape or meaning of something another party relies on (public signature, serialised format, column, event payload, default, ordering) without every consumer changing with it. Name the consumer. | A change to something shown to have no consumer outside the change. |

---

## Behaviour — what it does

### Functional

| Property | The question |
|---|---|
| `Correctness` | Matches its contract |
| `Completeness` | Every case in the spec handled |
| `Interoperability` | Speaks the protocol or standard it claims to |
| `Precision` | Numeric, temporal and encoding accuracy |

### Temporal

| Property | The question |
|---|---|
| `Determinism` | No hidden clock, ordering or randomness dependence |
| `Idempotency` | Retry and replay safe |
| `ConcurrencySafety` | Races, deadlocks, lost updates |
| `Ordering` | No undeclared sequencing requirements between calls |

### Data

| Property | The question |
|---|---|
| `Integrity` | Persisted state cannot become inconsistent |
| `Consistency` | The model holds across replicas and transactions |
| `Compatibility` | Backward and forward, wire and storage |
| `Durability` | Acknowledged writes survive |

## Architecture — how it is arranged

### Structure

| Property | The question |
|---|---|
| `Coupling` | Efferent, afferent, temporal, through shared data |
| `Cohesion` | The unit does one thing |
| `Placement` | Lives with its collaborators and its change-drivers |
| `Layering` | Dependency direction and inversion |
| `Boundaries` | Module, service, trust, transaction, consistency |
| `Granularity` | Sized right for its responsibility |

### Interface

| Property | The question |
|---|---|
| `Encapsulation` | Internals not leaking |
| `SignatureHonesty` | The type says what the function does |
| `AbstractionLevel` | No mixing, none premature |
| `DataModelling` | Illegal states unrepresentable, no primitive obsession |
| `StatePlacement` | Mutation and effects at the edge |

### Economy

| Property | The question |
|---|---|
| `Duplication` | Reimplementation of what exists |
| `Reuse` | Existing symbol available and ignored |
| `DependencyHygiene` | New third-party surface, direction, vendoring |
| `Deletability` | Can be removed later without surgery |

### Malleability

| Property | The question |
|---|---|
| `Changeability` | Cost of the next edit here |
| `Testability` | Exercisable without heroic setup |
| `Understandability` | A reader forms a correct model |
| `Analysability` | Reasonable about statically |
| `Extensibility` | The obvious next variant is cheap |
| `Portability` | Not bound to a host or runtime unnecessarily |

## Runtime — how it behaves under execution

### Efficiency

| Property | The question |
|---|---|
| `Performance` | Latency, throughput, complexity |
| `ResourceEfficiency` | Memory, handles, connections, allocations |
| `Cost` | Cloud spend as a first-class attribute |
| `Scalability` | Degrades linearly with load and data |

### Endurance

| Property | The question |
|---|---|
| `Reliability` | Failure rate under normal conditions |
| `Resilience` | Timeouts, retries, backpressure, circuit breaking |
| `Availability` | Single points of failure, blocking startup |
| `Recoverability` | Gets back to a good state after failure |
| `Degradation` | Fails partially rather than totally |

## Operations — how it is run

### Visibility

| Property | The question |
|---|---|
| `Observability` | The failure is diagnosable from what it emits |
| `Diagnosability` | Errors carry actionable context |
| `Traceability` | Request and decision provenance |
| `Auditability` | Who did what, retained |

### Control

| Property | The question |
|---|---|
| `Operability` | Can be run, tuned, intervened in |
| `Configurability` | Varies where it needs to, fixed where it does not |
| `Deployability` | No coordinated or manual release steps |
| `Migratability` | Schema and data changes have a rollout and rollback path |
| `Reversibility` | The change can be backed out |

## Risk

Kept separate because its suppression policy is usually set by different people, often a security team.

| Property | The question |
|---|---|
| `Security` | Confidentiality, integrity, authz, injection, secrets |
| `Privacy` | Personal data collected, logged, retained, or crossing a boundary |
| `Compliance` | Retention, residency, regulatory constraint |
| `Licensing` | Incompatible copied or vendored code |
| `Safety` | Physical or financial harm from misbehaviour |

## Systemic

Kept separate because its properties are emergent. It is a family like any other: a lens reading the whole context is what can see them.

| Property | The question |
|---|---|
| `Simplicity` | Accidental versus essential complexity |
| `ConceptualIntegrity` | One coherent idea, not several competing |
| `Symmetry` | Similar problems solved similarly |
| `DesignConsistency` | Matches the codebase's own patterns |
| `BlastRadius` | What fails architecturally when this fails |
| `Ownership` | Single writer, Conway alignment, who is on the hook |

---

## Notes on asking

- **Ask only what the context can answer.** No property may require running a build. `Ownership` needs CODEOWNERS and blame, and the across-time reading of `Duplication` (*the fourth near-duplicate validator this quarter*) needs history: use them only when you have read that input, and say so when you have not rather than guessing. `Duplication` and `Reuse` are only as good as your search for an equivalent: grep for it before reporting it.
- **Structure fires on movement, not level.** Plenty of modules sit at high coupling at rest. A metric is evidence only with a before and an after, so the question is always what this change moved.
- **An exclusion must not demand evidence a diff cannot hold.** *Do not report X unless the diff shows Y* silences the property when Y lives outside any diff by nature. For a bus consumer, redelivery is the broker's contract; for an HTTP handler, retries come from a client or a proxy. Where a platform fact is the trigger, name the observable that stands for it: the entry point's own kind is visible in the code.
- **Say what makes a property not its neighbour.** `Coupling` and `Layering`, or `Correctness` and `Precision`, share a lens; report a defect once, under the property that names it best.
- **The counter-example is worth most.** For each property, the sharpest move is naming what looks like this defect and is not.
