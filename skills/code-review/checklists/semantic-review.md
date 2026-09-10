# Semantic Review Checklist

Use this after automated checks and an initial diff read. Apply each section in proportion to the change's risk and the target repository's documented design. This is a prompt for investigation, not a mandate to report one finding per section.

## Responsibilities and architecture

- Does each module, service, or component have one coherent conceptual responsibility?
- Is ownership clear, including responsibilities spread across many individually small files or functions?
- Do dependencies point in the direction required by the project's architecture?
- Are domain/application decisions independent of transports and frameworks where the project expects that boundary?
- Are handlers/controllers coordinating inputs and outputs, or are they performing application/business work directly?
- Do application services depend on ports/contracts rather than concrete infrastructure when substitution or isolation is an intended boundary?
- Does the change reach across package or layer boundaries through private internals?
- Has convenience introduced accidental coupling, duplicate orchestration, or a hidden second owner for a policy?

Use Clean Architecture concepts pragmatically. A different intentional architecture is not a defect.

## Control flow and state

Prefer clarity over stylistic purity.

- Replace excessive nesting with guard clauses when branches are independent preconditions.
- Keep an ordinary `if` when the decision is simple and binary.
- For closed or discriminated states, prefer an exhaustive `switch` or the language's idiomatic pattern matching.
- For multi-step fallible pipelines, consider explicit typed result/state flow when it makes transitions and failures clearer.
- Avoid `else` after an unconditional return when removing it clarifies the happy path.
- Look for duplicated state transitions, implicit fallthrough, and loosely related booleans that permit invalid combinations.
- Question unconditional or infinite loops when a cursor, work item, or termination condition is already explicit.
- Do not remove readable conditionals merely to look functional, and do not add an FP library solely to eliminate them.
- Prefer compile-time exhaustiveness. Do not add artificial unreachable runtime branches only to appease a pattern or coverage tool.

## Semantic sources of truth

Check protocol values, statuses, error codes, route paths, media types, headers, limits, algorithms, state names, permission names, and enum-like strings.

- Does one semantic policy have one authoritative definition?
- Can copies drift independently across runtime behavior, schemas, tests, and generated clients?
- Is a local literal correctly local, or does it encode a shared contract?
- Are unrelated literals being centralized only because they happen to share a value?

Centralize meaning, not coincidental spelling.

## Errors and failure behavior

- Are error messages or string matching being used as control flow?
- Are sentinel values untyped or ambiguous?
- Can errors be swallowed, over-caught, or converted into successful empty results?
- Do infrastructure errors leak across application, API, or user-facing boundaries?
- Can stack traces, secrets, request contents, or internal details reach users or logs?
- Are retries bounded and classified by retryability?
- Is idempotency clear for retried or repeated operations?
- Is partial failure defined: what commits, what rolls back, and what can be retried?

Prefer typed/structured errors or explicit result states when the language and codebase support them.

## Data safety

Prioritize data safety over convenience.

- Can the change overwrite or delete existing data unexpectedly?
- Are deletion semantics, destructive defaults, and recovery expectations explicit?
- Is there a read-check-write sequence that assumes no concurrent writer?
- Are optimistic concurrency, version checks, conditional writes, transactions, or atomic operations needed?
- Are rollback/recovery and partial-update behavior appropriate for the storage boundary?
- Can user-controlled paths escape their intended root through traversal, encoding, links, or unsafe normalization?
- Are reads, writes, queues, collections, and payloads bounded where exhaustion is possible?

A race capable of silent user data loss is normally a blocker even when tests and CI pass.

## Security and privacy

- Are secrets present in code, configuration defaults, fixtures, errors, or logs?
- Could diagnostics expose bodies, notes, paths, identifiers, tokens, or other sensitive values?
- Did permissions or authorization scope become broader than required?
- Do defaults fail safely?
- Can a new path bypass authentication or authorization?
- Is data validated and normalized at the correct trust boundary?
- Is user-controlled content rendered, parsed, queried, or executed safely?
- In agent-facing systems, can untrusted prompt/content be interpreted as privileged instructions?
- Are framework, application, and infrastructure boundaries making conflicting trust assumptions?

Do not invent security requirements that the target system does not have; tie findings to an actual asset, boundary, or policy.

## Logging and observability

Scale expectations to operational needs and repository conventions.

- Is structured logging used where the project expects it instead of raw console output?
- Are sensitive payloads or forbidden concrete resource names/paths logged?
- Do events include enough operation, outcome, and correlation context to diagnose failures?
- Can expected failures be distinguished from defects or infrastructure failures?
- Is logging infrastructure leaking into domain/core code rather than being passed through an appropriate boundary?
- Are important state transitions or terminal failures invisible in an operationally significant path?

## Documentation

Review documentation quality, not mere presence.

- When required by project conventions, do public/exported APIs explain their purpose and contract?
- For non-obvious behavior, are invariants, units, side effects, failures, security implications, and lifecycle expectations documented as relevant?
- Do comments explain why rather than restate what the syntax already says?
- Are comments stale, misleading, or written as implementation-history/AI narrative rather than durable guidance?
- Is obvious trivial code being burdened with unnecessary documentation?

## Tests and test organization

- Do tests prove externally meaningful behavior, including edge, negative, and failure cases?
- Are concurrency or data-loss paths tested where risk warrants it?
- Do tests duplicate the implementation algorithm and therefore repeat the same bug?
- Are assertions brittle white-box checks rather than contract checks?
- Do tests only verify mocks, or do they establish the behavior and side effects that matter?
- Are mutations, emitted events, persistence, cleanup, and other side effects asserted?
- Are test suites labeled accurately?
  - **unit:** isolated behavior;
  - **integration:** intentional composition of multiple layers/components;
  - **E2E:** real externally meaningful boundaries or a true end-to-end path.
- Are test files included by the actual runner configuration, or merely present on disk?
- Does test placement follow the repository's established organization?

A dedicated `tests/` tree can improve navigation, but it is not a universal requirement. Prefer TDD when the project expects it, while reviewing the resulting behavior rather than trying to police commit chronology.

## Coverage

Treat coverage as a regression signal, not proof of quality. Inspect line, statement, function, and branch coverage where supported.

- Is relevant production source included even when no test imports it?
- Do exclusions hide difficult or newly added production code?
- Are new files uncovered or are risk-sensitive branches/functions untested?
- Were thresholds lowered without technical justification?
- Do new tests exercise behavior, or only game percentages?
- Is artificial unreachable runtime code creating meaningless branch misses?

Preserve an established healthy baseline; do not force meaningless 100% coverage.

## Dependencies and tooling

- Is each new dependency necessary, maintained, compatible, and already represented by an existing capability when possible?
- Is a deprecated or abandoned API used despite a supported alternative?
- Does the change add duplicate tooling or another package manager?
- Do commands bypass the canonical task runner or environment manager?
- Is lockfile churn generated by the intended package manager and scoped to the dependency change?
- Is version drift safe and intentional?
- Was a dependency added only to support a style preference?

Prefer the repository's existing conventions.

## Diagnostics beyond CI

Consider compiler, linter, formatter, deprecation, IDE/editor, framework, and generated API/schema diagnostics. Inspect actual repository configuration and whether CI runs the relevant modes. Zero CI errors does not establish zero meaningful diagnostics.

## Required final pass

Answer explicitly before concluding:

- Is the change actually correct?
- Is the architecture coherent?
- Are invariants preserved?
- Can it lose user data?
- Does failure behavior make sense?
- Did scope, access, or authority widen accidentally?
- Is it simpler than the viable alternatives?
- Are tests proving behavior rather than satisfying coverage?
