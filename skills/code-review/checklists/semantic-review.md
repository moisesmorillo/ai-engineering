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

## Dependency boundaries and semantic ownership

When the repository uses layered, Clean, Hexagonal, or ports-and-adapters architecture, verify dependency direction explicitly across domain/core, application/use cases, outbound ports, transport, infrastructure/adapters, and framework/composition. Inspect imports and contracts, not just directories. Ask which layer owns each type, which way the dependency points, whether the abstraction is expressed in application language, and whether a different adapter could implement it without awkward semantics.

The intended direction is inward: application/core may own the outbound ports it needs, while infrastructure implements them. An abstract persistence capability is valid and should not be flagged merely because it concerns storage:

```ts
interface ConditionalCurrentNoteRepository {
  read(...): ...
  create(...): ...
}
```

Flag instead when a port exposes R2/S3/Postgres objects, application receives storage ETags or bucket keys, core imports an adapter/framework module, or methods are shaped around SDK/framework operations rather than use-case needs. Application defines what capability it needs; infrastructure adapts to it.

Check for infrastructure vocabulary leakage even without a direct import. Terms such as `uploaded`, `bucket`, `object key`, `R2Object`, `customMetadata`, `wrangler`, `etag`, ORM/database rows, or filesystem paths can make an inner contract awkward if the adapter changes. Ask whether it would still make semantic sense after changing R2 to Postgres, S3, or a filesystem. Prefer `committedAt`, `persistedAt`, `generation`, `revision`, or an opaque replacement/CAS capability when those name the invariant; do not demand cosmetic renames when a term is genuinely storage-agnostic.

Keep persisted and framework representations at the boundary: JSON envelopes, `format: 2`, bridge-format markers, raw bytes, R2/custom metadata, storage ETags, SDK response types, Zod persistence schemas, Hono request/context types, Cloudflare Worker types, HTTP headers/statuses, and framework exceptions. Core/application should see application-level observations, receipts, timestamps, revisions, or opaque capabilities. Boundary code may depend inward; inner layers must not depend on framework APIs.

A type in `core/` is not automatically correctly owned, and an adapter-private type may legitimately reference application types. Findings must identify the inner location, leaked outer concern, dependency direction, coupling/adaptor constraint, recommended owner, port/DTO move or rename, and required tests/type checks. Use MINOR/NOTE for vocabulary coupling without a concrete dependency, MAJOR for concrete infrastructure/framework or persisted-representation leakage, and BLOCKER only for an already demonstrated correctness, security, or data-integrity failure.

- [ ] Dependency direction follows the intended architecture: inner layers do not import concrete transport, framework, storage, or adapter modules.
- [ ] Outbound ports are owned by the application/core and describe use-case needs, not SDK/framework operations.
- [ ] Persisted/serialized representation details remain in adapters/infrastructure; core sees application-level observations/capabilities only.
- [ ] Inner-layer vocabulary remains adapter-agnostic where practical; changing the concrete adapter would not make core contracts semantically awkward.

## Repository-wide duplication and reuse

Treat reuse as a correctness and ownership question, not a blanket DRY rule. For every changed or newly introduced protocol, business, storage, security, or lifecycle concept, search the whole repository—not just changed files—for:

- the same and related function/helper names, type names, constants, and enum-like values;
- literals, route strings, regexes, schemas, serialization formats, media types, and headers; and
- error/status codes, storage prefixes, retry/CAS terms, and business-policy wording.

Classify matches before recommending action:

- **Textual duplication:** similar or identical code. Usually low severity unless it encodes one shared rule.
- **Structural duplication:** different shapes implementing the same workflow or transformation. Investigate whether they can drift.
- **Semantic duplication:** multiple owners of one business, protocol, security, storage, concurrency, or lifecycle invariant. This is the highest-value category.

Ask whether the new code reuses an existing primitive or creates a second source of truth. Check for existing parsers/formatters, validators/schemas, route capability tables, policy services, ports, storage-key helpers, and error/status mappings before accepting a new helper, type, regex, or constant. Ask what layer should own the knowledge and whether callers are re-implementing it across architecture boundaries.

Do not recommend an abstraction merely because code looks alike. Consolidation needs at least one of: one semantic rule, credible behavioral drift, security/data-safety significance, likely frequent change, or a clear architectural owner. Keep local duplication when values only coincide, a transformation is incidental, or callers intentionally have different policies. Do not report a finding solely to satisfy this checklist.

A duplication finding must name exact duplicated locations, the semantic rule, how the copies can drift, the source-of-truth owner, what to reuse or centralize, what not to generalize, and tests for the consolidation and boundary behavior. Typical severity is MINOR or a review NOTE for low-drift duplication; use MAJOR for multiple owners of auth, protocol acceptance, destructive operations, CAS/concurrency, storage formats, routes/OpenAPI/CORS, lifecycle/recovery, retry, or idempotency; use BLOCKER only for a concrete correctness or security defect already caused by the copies.

- [ ] New code reuses existing repository primitives where semantics match; it does not introduce parallel validators/parsers/helpers for an existing rule.
- [ ] High-value semantic rules have one authoritative source of truth; routing, protocol formats, auth, storage layout, state transitions, and error mappings are not independently duplicated across layers.
- [ ] Repository-wide search was performed for changed protocol/business concepts, not only diff-local inspection.

## Control-flow complexity and state

Prefer clarity over stylistic purity. Treat cyclomatic complexity (independent paths), cognitive complexity (nesting and mental breaks), nesting depth, boolean-expression length, repeated guards, and implicit transitions as evidence to investigate—not automatic defects. Use configured metrics when available; otherwise reason qualitatively. Ask whether the paths make behavior exhaustive, invalid states visible, security invariants auditable, tests mappable to decisions, and later changes safe.

- Replace excessive nesting with guard clauses when branches are independent preconditions.
- Keep an ordinary `if` when the decision is simple and binary; several short, obvious early returns may be the clearest implementation.
- For closed or discriminated states, prefer an exhaustive `switch` or idiomatic pattern matching when it makes coverage and invalid states obvious.
- Look for one function mixing classification, combination/conflict validation, policy, variant parsing, and domain construction; hidden state-machine transitions; loosely related booleans that permit invalid combinations; and duplicated parsing/validation logic.
- When a closed protocol/state matrix has materially difficult combinations, consider `classify -> exhaustive dispatch -> variant-specific handling`, or an explicit decision table/state-machine equivalent. Do not impose this shape where a few guards are simpler.
- For multi-step fallible pipelines, consider explicit typed result/state flow when it makes transitions and failures clearer.
- Question unconditional or infinite loops when a cursor, work item, or termination condition is already explicit.
- Do not remove readable conditionals merely to look functional, prescribe a switch for its own sake, or add an FP library solely to eliminate them. Prefer compile-time exhaustiveness; do not add artificial unreachable runtime branches only to appease a pattern or coverage tool.

Raise a complexity concern from a MINOR/non-blocking note to MAJOR only when it materially increases correctness risk in authorization, destructive operations, concurrency/CAS, retries/idempotency, lifecycle/state transitions, recovery/deletion flows, or protocol acceptance. Complexity alone is not a BLOCKER. A finding must name the exact function/file, explain the decision matrix and why it matters, identify mixed responsibilities, propose a lower-complexity shape, say whether behavior can remain unchanged, and name regression tests. High coverage does not automatically excuse hard-to-audit branch interactions: check whether tests map to decisions and whether mutation testing could expose gaps. Conversely, do not manufacture a finding when control flow and tests make completeness obvious.

- [ ] Control flow is auditable: high-risk parsers/state machines/auth/concurrency code does not hide a large decision matrix inside ad-hoc branching; closed states and exhaustive dispatch are used where they materially reduce reasoning complexity.

## Lifecycle and concurrency state ownership

For code with async work, background work, retries, unload/reload, enable/disable, or other lifecycle transitions, identify each relevant lifetime explicitly (for example, operation, session/component, plugin, or process). For every `busy`, `pending`, `inFlight`, `loaded`, `active`, or similar guard, ask: **what lifetime owns this state, and what happens if that lifetime ends before the operation it guards?** Then verify:

- the guard belongs to the lifetime of the invariant it protects, rather than merely the nearest object or session;
- teardown, reload, re-enable, and retry cannot reset exclusion while non-cancellable work is still pending;
- cancellation is assumed only when the host API explicitly guarantees it;
- stale completion cannot mutate or present through a newer lifetime; and
- operation exclusion is separate from presentation/session identity when those concerns have different lifetimes.

## Semantic sources of truth

Check protocol values, statuses, error codes, route paths, media types, headers, limits, algorithms, state names, permission names, and enum-like strings.

- Does one semantic policy have one authoritative definition?
- Can copies drift independently across runtime behavior, schemas, tests, and generated clients?
- Is a local literal correctly local, or does it encode a shared contract?
- Are unrelated literals being centralized only because they happen to share a value?

Centralize meaning, not coincidental spelling.

- [ ] Structured protocol formats have a canonical parser/formatter; callers do not validate a representation and then manually slice, split, or regex that same representation.

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
- For lifecycle or concurrency behavior, reproduce the harmful interleaving, not merely the same operations in a safe order. Keep the original operation pending while triggering the competing action or lifecycle transition; assert that the competing operation does not start, that stale completion cannot affect the new lifetime, and that normal operation resumes only after the original work settles.
- Read the ordering of awaits, resolve/reject calls, teardown, and re-enable closely. Confirm the test reaches the failure state before its assertion boundary; do not infer behavioral coverage from a nearby test name or high branch coverage.

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
