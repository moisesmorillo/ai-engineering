---
name: code-review
description: Performs evidence-based semantic code and pull-request review beyond CI, linting, formatting, and typechecking. Use to assess correctness, architecture, maintainability, data safety, security, operability, tests, coverage, and TypeScript-specific risks, including corrective re-reviews after changes.
license: MIT
---

# Code Review

Review whether a change is safe, correct, coherent, and maintainable—not merely whether automation passes. Green CI is necessary evidence, but it is not proof that a change should ship.

## Authority and scope

After discovering the target and inspecting its complete diff, locate the target repository's applicable `AGENTS.md`, contributing guide, architecture documents, ADRs, milestone/spec/plan, CI configuration, tests, and nearby established conventions. Let changed concepts and affected paths guide this search; do not read every document blindly. Repository evidence overrides this skill's defaults. Do not mechanically impose Clean Architecture, file layouts, test locations, documentation formats, aliases, or tools on a project that intentionally uses another approach.

Keep findings within the change's risk surface and accepted delivery boundary. Inspect surrounding code and search repository-wide for semantic owners when needed to establish a contract or invariant, but do not turn a focused review into an unrelated redesign or report explicitly deferred later-phase behavior as missing.

## Normal invocation

The intended day-to-day invocation is simply:

```text
/skill:code-review
```

The reviewer must autonomously discover the active repository, change or PR, correct baseline, relevant repository context, risk dimensions, and canonical validation. The user should not normally need to provide a repository URL, PR number, base SHA, milestone, slice, mini review brief, or prompts to check inferable concerns such as effect certainty, architecture boundaries, protocol validation, ADR compliance, lifecycle races, or semantic reuse.

An explicit user target, range, repository, context, or focus is an override or additive focus, not a prerequisite and not a replacement for autonomous discovery. If the environment genuinely cannot identify a safe target or baseline, ask one concise question for only the missing information rather than guessing.

## Review workflow

Run [`checklists/review-discovery.md`](checklists/review-discovery.md) first; it is the single authoritative algorithm for target, baseline, context, and internal Review Brief discovery. Then use [`checklists/pr-review.md`](checklists/pr-review.md) for the ordered change/PR workflow and [`checklists/semantic-review.md`](checklists/semantic-review.md) for the final language-agnostic semantic pass. When the changed code is TypeScript, also load [`profiles/typescript.md`](profiles/typescript.md). See [`examples/discovery.md`](examples/discovery.md) for one-command discovery scenarios.

At minimum:

1. Discover the repository, complete review target, correct baseline/range, and any intentional working-tree overlay.
2. Inspect the complete diff, identify changed concepts, and derive an evidence-backed internal Review Brief from relevant repository sources.
3. Inspect automated checks, canonical validation, and meaningful diagnostics.
4. Read affected implementation, tests, semantic owners, and surrounding contracts.
5. Review architecture, control flow, failure behavior, data safety, security, lifecycle/concurrency, operability, and compatibility in proportion to discovered risk.
6. Review test semantics and coverage—not just test counts or percentages.
7. Perform a final manual semantic pass after automation.
8. Report only specific, evidenced, actionable findings.
9. On corrective review, recover and verify every prior finding individually while checking the fix for regressions.

## Core decision heuristics

- Prefer conceptual responsibility and clear ownership over arbitrary file or function size limits.
- Preserve the project's intentional architecture; flag boundary erosion and accidental dependency direction, not pattern differences by themselves.
- Prefer clear control flow: guard independent preconditions, use ordinary `if` for simple binary choices, and use exhaustive handling for closed states. Treat control-flow complexity as a correctness and auditability signal, not a branch-count style rule; apply the detailed control-flow checks below.
- Give one semantic policy one authoritative source. Do not centralize unrelated literals merely because their text or value matches; apply the repository-wide duplication and reuse audit below rather than a blanket DRY rule.
- Prefer typed or structured failures when supported. Never let an important failure silently become a successful empty result.
- Prioritize data integrity, trust boundaries, and externally observable behavior over convenience or style.
- Treat coverage as a regression signal, not proof of behavior quality.
- For async work that crosses lifecycle boundaries, identify the relevant lifetimes and make each `busy`, `pending`, `inFlight`, `loaded`, or `active` guard belong to the lifetime of the invariant it protects. Do not assume teardown cancels host work; check that reload, re-enable, or retry cannot reset exclusion while non-cancellable work remains pending, and that stale completion cannot affect a newer lifetime. When operation ownership and presentation/session ownership differ, keep them separate. Apply the detailed lifecycle/interleaving checks in the semantic checklist.
- Do not add dependencies, abstraction, functional-programming libraries, or type cleverness solely to satisfy reviewer taste.

## Repository-wide duplication and reuse audit

This is a semantic ownership audit, not a request to eliminate repeated text. Review the whole repository for missed reuse and independently editable owners of the same knowledge.

### Classify the duplication

- **Textual duplication:** similar or identical code, literals, or names. This is usually low severity unless the text encodes one shared rule.
- **Structural duplication:** different code shapes implement the same workflow or transformation. It is important when the workflows can diverge, even if no lines match.
- **Semantic duplication:** multiple locations encode the same business, protocol, security, storage, concurrency, or lifecycle invariant. This is the highest-value category.

Examples of semantic duplication include route definitions repeated in router, CORS, and OpenAPI; ETag parsing repeated in transport and core; auth checks copied across handlers; storage-key construction repeated in adapters; status/error mapping implemented independently; lifecycle transitions copied between services; regexes defining one identifier format; and retry/CAS/effect-certainty logic implemented more than once.

### Search repository-wide and identify ownership

When a changed or newly introduced concept is found, search the repository—not only changed files or the PR diff—for:

- function and helper names, related type names, constants, and enum-like values;
- literals, route strings, regexes, schema definitions, serialization formats, media types, and headers;
- error/status codes, storage prefixes, retry/CAS terms, and business-policy wording.

Determine whether the change reuses an existing primitive or creates a second source of truth. Ask what should own the knowledge, whether an authoritative layer already exists, and whether callers are re-implementing knowledge that belongs to a parser, formatter, schema, port, policy service, route capability table, or storage-key helper. A new UUID validator, digest helper, path encoder, result/error type, or retry helper requires evidence that the existing primitive has different semantics.

Raise a finding when one semantic rule has multiple independently editable owners and a future change to one copy can change behavior without changing the other. Prefer one canonical parser/formatter, route policy, schema/validator, business-policy service, storage-key helper, or error/status mapping owner. Do not centralize unrelated values merely because strings or shapes happen to match. If callers intentionally have different policies, preserve local implementations and explain the semantic distinction.

### Abstraction threshold and finding requirements

Do not recommend a shared abstraction merely because two blocks look alike. Recommend reuse or consolidation only when at least one is true:

- the code represents one semantic rule;
- independent copies can cause behavioral drift;
- it is security- or data-safety-critical;
- change frequency makes divergence likely; or
- the abstraction has a clear architectural owner.

Prefer local duplication over a misleading abstraction for coincidentally identical literals, tiny incidental transformations, or callers with intentionally different policies. Reuse guidance remains subordinate to correctness, clear ownership, and the project's architecture.

Every duplication finding must include: (1) exact duplicated locations, including the existing owner or competing implementation; (2) the semantic rule being duplicated; (3) why the copies can drift; (4) the recommended owner/source of truth; (5) what should be reused or centralized; (6) what must not be generalized because semantics differ; and (7) tests that protect the consolidation and the relevant boundary behavior. Never create a finding solely to satisfy this audit.

### Duplication severity

Use demonstrated impact, not the amount of repeated code. A low-risk duplicated implementation is usually a review NOTE/NIT or MINOR. Multiple sources of truth for auth, protocol acceptance, destructive operations, CAS/concurrency, storage formats, routes/OpenAPI/CORS, recovery/lifecycle policy, retry, or idempotency are typically MAJOR. Use BLOCKER only when the duplication already produces a concrete correctness or security defect, such as one copy accepting data another rejects or a boundary being bypassed.

## Dependency boundaries and architecture ownership

For repositories using layered, Clean, Hexagonal, or ports-and-adapters architecture, verify dependency direction explicitly. Do not impose one of these architectures on a repository that has intentionally chosen another design, and do not treat every persistence, storage, or network reference as a violation.

Inspect imports and semantic ownership across domain/core, application/use cases, outbound ports, transport, infrastructure/adapters, and framework/composition. Ask:

- Which layer owns this type or interface, and which way does the import point?
- Is an inner layer depending on an outer-layer implementation detail?
- Is the abstraction expressed in application language or in the current adapter's language?
- Could another adapter implement the port without awkward semantics?
- Does the file's location agree with its meaning, or is a supposedly core type actually a transport/storage representation?

The dependency rule is inward: application/domain/core may define the outbound ports they need, and infrastructure may implement those ports. It is correct for application to own an abstract persistence capability, for example:

```ts
interface ConditionalCurrentNoteRepository {
  read(...): ...
  create(...): ...
}
```

The rule is **application defines what capability it needs; infrastructure adapts to it**. Do not flag a port merely because it concerns persistence. Flag the concrete dependency instead: core importing an adapter or framework module; a port exposing R2/S3/Postgres objects; application receiving storage ETags, bucket keys, or custom metadata; or methods shaped around an SDK/framework API rather than use-case needs.

### Leakage heuristics

Inspect inner-layer vocabulary even when imports point inward correctly. Terms such as `uploaded`, `bucket`, `object key`, `R2Object`, `customMetadata`, `wrangler`, `etag`, `Hono Context`, ORM/database rows, or filesystem-path semantics may make a port or application contract depend conceptually on one adapter. Ask whether the type would still make semantic sense if the adapter changed from R2 to Postgres, S3, or a filesystem. If not, raise a finding; prefer application language such as `committedAt`, `persistedAt`, `generation`, `revision`, or an opaque replacement capability when that names the actual invariant. Do not demand a rename when the term is genuinely storage-agnostic or when only style changes.

Persisted and transport representations should remain at the boundary. Keep JSON envelope shapes, storage-format discriminators such as `format: 2`, bridge-format markers, raw object bodies, R2/custom metadata, bucket/object ETags, storage SDK responses, Zod persistence schemas, HTTP headers/statuses, Hono request/context types, Cloudflare Worker types, and framework exceptions in infrastructure or transport. Core/application should receive application-level observations, receipts, authoritative timestamps, revisions, or opaque CAS/replacement capabilities instead. Boundary code may depend inward; inner layers should not depend on framework APIs or use framework exceptions as application flow control.

Review semantic ownership rather than folder purity. A type in `core/` is not automatically correct, and an adapter-private type in `infrastructure/` may legitimately reference application types. Cross-reference the reuse/source-of-truth audit when a leaked representation creates a second owner, and the protocol-format guidance when headers, ETags, or serialization rules cross the boundary.

### Architecture finding quality and severity

A dependency-boundary finding must include: (1) the inner-layer location; (2) the outer-layer concern leaking inward; (3) the dependency direction; (4) why it increases coupling or constrains adapters; (5) the recommended owner; (6) whether a port/DTO should move or be renamed; and (7) tests or type checks needed after the change. Avoid "violates Clean Architecture" without evidence.

Use a review NOTE/NIT or MINOR for adapter-flavored vocabulary that does not create a concrete dependency. MAJOR is appropriate when application/core imports infrastructure or framework modules, core contracts expose concrete SDK types, business policy depends on transport/storage details, or persisted representations leak inward and constrain future adapters. Use BLOCKER only when the violation already causes a concrete correctness, security, or data-integrity failure.

## Control-flow complexity and format ownership

Evaluate whether independent execution paths make behavior, invalid states, security invariants, test coverage, or future changes hard to reason about. Where tooling provides cyclomatic or cognitive-complexity metrics, inspect them; otherwise estimate qualitatively from independent conditions, boolean operators, nesting, repeated guards, and state transitions. Do not introduce tooling or dependencies solely to obtain a number unless requested. Complexity is one signal alongside correctness, architecture, security, tests, performance, and maintainability—not a target number.

Inspect especially validators, consistency checkers, protocol/token classifiers and parsers, authorization and permission state, concurrency/CAS and synchronization, retries/effect certainty and idempotency, lifecycle transitions, configuration state, recovery/tombstone/deletion flows, and other state machines for hidden decision matrices, including:

- multiple independent invariant families owned by one function;
- repeated inspection of the same discriminant across separate branches;
- mutually exclusive closed variants encoded through negative checks, priority, fallthrough, or a final catch-all path rather than explicit exhaustive handling;
- business or state-machine compatibility tables encoded as condition order;
- global invariants mixed with per-entity, per-item, or per-path invariants;
- logic that requires the reader to reconstruct valid state/action combinations mentally;
- booleans that irreversibly collapse materially different invalid states before callers, tests, migrations, or operations can use the reason;
- one function mixing classification, validation, policy, variant parsing, and domain-object construction; and
- tests that cover lines or isolated cases without mapping clearly to valid and invalid combinations.

Prefer a finding when one function owns several semantically distinct invariant families; a closed union is handled implicitly and policy completeness is obscured; conditions form a state/action compatibility matrix; adding a variant would require edits in several distant branches that are easy to miss; security-, data-safety-, protocol-, or lifecycle-critical policy is materially harder to audit; or operationally important failure reasons are collapsed too early. Prefer no finding when guards are a short linear precondition list, all conditions express one semantic rule, exhaustive structure would add verbosity without exposing policy, or helper extraction would only fragment readable local logic.

### Preferred structural patterns

Decompose by semantic invariant ownership, not by branch count. For example:

```text
validateDeviceState()
  ├─ validateLifecycle()
  ├─ validateIdentifierUniqueness()
  ├─ validateHandoff()
  ├─ validateDrainedState()
  └─ validatePathState()
       ├─ validateUnresolvedMutation()
       └─ validateDesiredState()
```

For a closed protocol or state matrix, consider `classify -> exhaustive dispatch -> focused variant validation`, an explicit decision table, or another representation that makes completeness visible. An exhaustive `switch` can be appropriate when each discriminated-union variant has different rules:

```ts
switch (ack.kind) {
  case "unassociated":
  case "live":
  case "tombstone":
}
```

Do not prescribe `switch` when a map, table, classifier, or a few guards are clearer. Do not create one helper per `if`, introduce a generic validation framework, require a `Result` or error type for a trivial predicate, or add abstraction merely to reduce visible complexity.

When invalid-state identity materially improves corrupt-state diagnosis, tests, operational debugging, migrations, or lifecycle work, consider an internal diagnostic result while retaining a public boolean facade where useful:

```ts
type ValidationResult =
  | { kind: "valid" }
  | { kind: "invalid"; reason: ValidationFailureReason };
```

Do not require diagnostic validation for every validator. The finding must identify who can use the reason and why losing it matters.

### Finding and severity calibration

Do not flag branch count mechanically. Several obvious early-return guards, a short exhaustive switch over a discriminated union, and straightforward per-condition validation can be preferable to abstraction.

- **NOTE/NIT:** local readability issue without meaningful drift or audit risk.
- **MINOR:** a hidden decision matrix or mixed invariant ownership makes future changes error-prone, but current behavior appears correct.
- **MAJOR:** implicit state handling creates a credible correctness, safety, security, lifecycle, protocol, or drift risk—for example, a variant can be silently omitted or conflicting policies are encoded in distant branches.
- **BLOCKER:** only when there is a concrete severe defect; complexity alone is never a blocker.

A complexity finding must include the severity, exact function/file, why the paths increase reasoning risk, the mixed responsibilities, a concrete lower-complexity shape, whether behavior can remain unchanged, and tests that should protect the refactor. Explain the failure or audit scenario; never write only “too many if statements.” High coverage does not excuse a decision matrix when interactions remain hard to audit: ask whether tests map to its decisions and whether mutation testing could expose untested branch interactions. Conversely, do not demand refactoring when simple control flow and tests make completeness obvious.

If a caller validates a structured identifier or protocol representation and then manually `slice`, `split`, or regexes that same representation, flag duplicated format knowledge when it can drift. Prefer one canonical parser/formatter pair at the owning layer (for example, ETags, encoded IDs, cursors, versioned envelopes, or protocol tokens); preserve behavior while moving extraction behind that owner.

## Severity and prioritization

Assign severity from demonstrated impact and likelihood, not the amount of code needed to fix it.

### BLOCKER

Concrete evidence shows the change cannot safely ship in the target release. Typical cases include likely irreversible data loss, an exploitable security-boundary failure, a hidden unsafe-mutation race, failure of required build/CI/tests, a required external contract break with no safe mitigation, or violation of an invariant essential to safe operation. Not every behavior bug is a BLOCKER; state the release-stopping scenario.

### MAJOR

A meaningful correctness or engineering defect that should normally be fixed before approval, but whose demonstrated impact is not release-stopping: architecture erosion, material type unsafety, an unhandled important failure mode, incorrect externally observable behavior, a misleading protocol/API contract, a significant maintainability problem, or missing tests for high-risk behavior. Escalate to BLOCKER only when concrete impact makes the target release unsafe.

### MINOR

A localized issue with limited impact: maintainability or organization problems, weak documentation, modest test gaps, naming inconsistencies, or small avoidable complexity. It should be addressed, but need not block when the repository permits follow-up.

### NIT / OPTIONAL

A preference-level improvement with no meaningful correctness, safety, contract, or maintainability impact. Use sparingly. Never block a PR on style preference when behavior, architecture, and repository conventions are sound.

## Severity, disposition, and verdict

Severity describes impact; disposition describes whether the finding must be resolved before approval. Do not use “blocking” as a synonym for the `BLOCKER` severity:

- `BLOCKER` findings always block approval.
- `MAJOR` findings normally block approval. An owner may explicitly accept one only when repository policy permits it and the residual risk is bounded and documented.
- `MINOR` findings are normally non-blocking, but may block when an explicit acceptance criterion or repository rule requires correction.
- `NIT / OPTIONAL` findings never block approval.

Use `REQUEST CHANGES` while any finding blocks approval. Use `APPROVE WITH NOTES` when only acknowledged, non-blocking findings remain. Use `APPROVE` when no actionable finding remains.

A finding is blocking only with credible evidence that it violates a required acceptance criterion or that merging would create material correctness, safety, contract, or project-invariant risk. State the failure scenario explicitly. A non-blocking finding is a bounded improvement whose deferral does not materially increase shipping risk.

Uncertainty alone does not block approval. Investigate the relevant contract, ask a focused question, or state what evidence is missing. Do not invent bugs from stylistic discomfort.

## Finding quality

Every finding must be:

- **specific:** identify one concrete problem;
- **evidenced:** cite the relevant location and behavior or contract;
- **actionable:** recommend a direction without prescribing needless implementation detail;
- **scoped:** explain the affected path, state, user, or boundary;
- **prioritized:** assign severity based on impact.

Avoid comments such as "could be cleaner," "not best practice," or "consider refactoring" without a demonstrated consequence. Examples of weak and strong findings are in [`examples/findings.md`](examples/findings.md).

If no actionable finding exists, explicitly approve. Do not manufacture findings to make the review appear thorough.

## Default output

The Review Brief remains internal. Include the optional short `Review context` only when it helps the reader understand the target, governing evidence, intentional scope boundary, or principal risk; do not emit a long preliminary discovery report.

```text
Verdict: REQUEST CHANGES / APPROVE WITH NOTES / APPROVE

Review context (optional)
- target: PR/branch/range and any intentional working-tree overlay
- governing evidence: relevant spec/ADR/contract and explicit scope boundary
- principal risks: only the highest-value discovered review dimensions

Blocking findings

[SEVERITY] Concise title
Location: path/file.ext:line
Why it matters: Concrete impact and affected scenario.
Evidence: Relevant code path, contract, diagnostic, or reproducer.
Recommended direction: Bounded remediation direction.

Non-blocking findings

[SEVERITY] Concise title
Location: path/file.ext:line
Why it matters: Bounded impact.
Evidence: Relevant code path, contract, or diagnostic.
Recommended direction: Bounded remediation direction.

Validation reviewed
- CI: status and scope inspected
- tests: relevant suites and behavior inspected
- coverage: change and configuration inspected
- lint/typecheck/diagnostics: results and gaps inspected
- manual semantic review: completed

Final assessment
One short paragraph explaining the verdict and residual risk.
```

Replace `[SEVERITY]` with `BLOCKER`, `MAJOR`, `MINOR`, `NIT`, or `OPTIONAL`; place the finding by disposition rather than severity alone. Omit empty sections. Distinguish checks personally run from checks only observed. Do not claim validation that was not performed.

## Final semantic questions

Before issuing the verdict, ask:

- Is this actually correct for every relevant state and failure path?
- Is the architecture still coherent and is dependency direction preserved?
- Are contracts and invariants preserved?
- Can this lose, overwrite, expose, or corrupt user data?
- Does error behavior make sense at each boundary?
- Did the change accidentally widen scope or permissions?
- Is the implementation simpler or more complicated than necessary?
- Are tests proving behavior or only satisfying mocks and coverage?

For a re-review, automatically rediscover the same active change, recover prior findings from current conversational context when available, and retain the original baseline when it remains valid. Do not require the user to paste the original brief or findings again. Rebuild the Review Brief if the complete diff, governing evidence, scope, or risk materially changed. Do not say "looks good" merely because CI is green: inspect the corrective delta and current complete change, and mark each prior finding `fixed`, `partially fixed`, `not fixed`, `explicitly deferred with acceptable rationale`, or `withdrawn / not applicable`, with evidence supporting that status. A withdrawal must cite evidence disproving the original finding. For lifecycle or concurrency fixes, passing the original scenario is not enough: verify that state and responsibility moved to the correct lifetime and layer, that the fix is not merely another flag in the wrong abstraction, that it introduces no stale-state or reset regression, that the invariant is owned by the narrowest correct layer, and that tests exercise the exact original failure interleaving. Approve only when every remaining issue is truly non-blocking.
