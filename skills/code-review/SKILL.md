---
name: code-review
description: Performs evidence-based semantic code and pull-request review beyond CI, linting, formatting, and typechecking. Use to assess correctness, architecture, maintainability, data safety, security, operability, tests, coverage, and TypeScript-specific risks, including corrective re-reviews after changes.
license: MIT
---

# Code Review

Review whether a change is safe, correct, coherent, and maintainable—not merely whether automation passes. Green CI is necessary evidence, but it is not proof that a change should ship.

## Authority and scope

Before reviewing, read the target repository's applicable `AGENTS.md`, contributing guide, architecture documents, ADRs, CI configuration, and nearby established conventions. Those sources override this skill's defaults. Do not mechanically impose Clean Architecture, file layouts, test locations, documentation formats, aliases, or tools on a project that intentionally uses another approach.

Keep findings within the change's risk surface. Inspect surrounding code when needed to establish a contract or invariant, but do not turn a focused review into an unrelated redesign.

## Review workflow

Use [`checklists/pr-review.md`](checklists/pr-review.md) for the ordered PR workflow. Use [`checklists/semantic-review.md`](checklists/semantic-review.md) for the final language-agnostic semantic pass. When the changed code is TypeScript, also load [`profiles/typescript.md`](profiles/typescript.md).

At minimum:

1. Establish local rules, intended behavior, acceptance criteria, and risk.
2. Inspect automated checks and meaningful diagnostics.
3. Read the actual diff, affected implementation, and affected tests.
4. Review architecture, control flow, failure behavior, data safety, security, operability, and compatibility.
5. Review test semantics and coverage—not just test counts or percentages.
6. Perform a final manual semantic pass after automation.
7. Report only specific, evidenced, actionable findings.
8. On corrective review, verify every prior finding individually.

## Core decision heuristics

- Prefer conceptual responsibility and clear ownership over arbitrary file or function size limits.
- Preserve the project's intentional architecture; flag boundary erosion and accidental dependency direction, not pattern differences by themselves.
- Prefer clear control flow: guard independent preconditions, use ordinary `if` for simple binary choices, and use exhaustive handling for closed states. Treat control-flow complexity as a correctness and auditability signal, not a branch-count style rule; apply the detailed control-flow checks below.
- Give one semantic policy one authoritative source. Do not centralize unrelated literals merely because their text or value matches.
- Prefer typed or structured failures when supported. Never let an important failure silently become a successful empty result.
- Prioritize data integrity, trust boundaries, and externally observable behavior over convenience or style.
- Treat coverage as a regression signal, not proof of behavior quality.
- For async work that crosses lifecycle boundaries, identify the relevant lifetimes and make each `busy`, `pending`, `inFlight`, `loaded`, or `active` guard belong to the lifetime of the invariant it protects. Do not assume teardown cancels host work; check that reload, re-enable, or retry cannot reset exclusion while non-cancellable work remains pending, and that stale completion cannot affect a newer lifetime. When operation ownership and presentation/session ownership differ, keep them separate. Apply the detailed lifecycle/interleaving checks in the semantic checklist.
- Do not add dependencies, abstraction, functional-programming libraries, or type cleverness solely to satisfy reviewer taste.

## Control-flow complexity and format ownership

Evaluate whether independent execution paths make behavior, invalid states, security invariants, test coverage, or future changes hard to reason about. Where tooling provides cyclomatic or cognitive-complexity metrics, inspect them; otherwise estimate qualitatively from independent conditions, boolean operators, nesting, repeated guards, and state transitions. Do not introduce tooling or dependencies solely to obtain a number unless requested. Complexity is one signal alongside correctness, architecture, security, tests, performance, and maintainability—not a target number.

Inspect especially protocol/token parsers (including conditional headers), authorization and permission state, concurrency/CAS and synchronization, retries/effect certainty and idempotency, lifecycle transitions, configuration state, recovery/tombstone/deletion flows, and other state machines for:

- long boolean expressions or repeated conditions whose combinations form a decision matrix;
- one function mixing classification, validation, policy, variant parsing, and domain-object construction;
- implicit state-machine transitions, loosely related booleans, or duplicated parsing/validation logic; and
- tests that cover lines or individual cases without mapping clearly to valid and invalid combinations.

Do not flag branch count mechanically. Several obvious early-return guards, a short exhaustive switch over a discriminated union, and straightforward per-condition validation can be preferable to abstraction. For a closed protocol or state matrix, consider `classify -> exhaustive dispatch -> variant-specific handling`, or an equivalent explicit state-machine/decision table. Prefer discriminated unions and exhaustive switches when they make completeness and invalid states materially more visible; do not prescribe this shape when a few guards are simpler.

Complexity alone is usually a non-blocking MINOR or review NOTE. Raise it to MAJOR when the structure materially increases the chance of an unreviewed correctness or security gap in authorization, destructive operations, concurrency/CAS, retries/idempotency, lifecycle/state transitions, recovery/deletion, or protocol acceptance. Do not use BLOCKER solely for complexity without a concrete correctness or security failure.

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

```text
Verdict: REQUEST CHANGES / APPROVE WITH NOTES / APPROVE

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

For a re-review, do not say "looks good" merely because CI is green. Mark each prior finding `fixed`, `partially fixed`, `not fixed`, `explicitly deferred with acceptable rationale`, or `withdrawn / not applicable`, with evidence supporting that status. A withdrawal must cite evidence disproving the original finding. For lifecycle or concurrency fixes, passing the original scenario is not enough: verify that state and responsibility moved to the correct lifetime and layer, that the fix is not merely another flag in the wrong abstraction, that it introduces no stale-state or reset regression, that the invariant is owned by the narrowest correct layer, and that tests exercise the exact original failure interleaving. Approve only when every remaining issue is truly non-blocking.
