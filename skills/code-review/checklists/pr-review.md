# Change and Pull Request Review Workflow

Follow this order for an initial review of a PR, branch, commit range, working-tree change, or explicit repository-wide audit. Increase depth for high-risk changes; do not skip the final semantic pass because automation is green.

## 1. Discover the review target and context

Run [`review-discovery.md`](review-discovery.md) before evaluating findings, including for a no-argument `/skill:code-review` invocation. Resolve the repository, target, baseline/range, intentional local overlay, governing evidence, accepted scope, semantic owners, risk profile, and canonical validation into the internal Review Brief. An explicit user target overrides automatic target selection; user focus remains additive to autonomous discovery.

If discovery leaves materially different targets or baselines plausible, ask one concise question and stop before findings. Do not guess. Keep the detailed precedence and evidence-conflict algorithm owned by the discovery checklist rather than duplicating it here.

## 2. Confirm the complete diff and intended change

Read every production, test, configuration, schema, generated, dependency, and documentation file in the Review Brief's complete range, including any intentionally included working-tree overlay. Watch for scope creep, generated or lockfile churn, hidden deletions, changed defaults, and behavior split across commits or files. Do not review only the latest commit of a multi-commit change.

Confirm the brief's intent against the actual diff, applicable repository instructions, current contracts/design evidence, tests, and established conventions. Treat PR/issue descriptions as supporting evidence rather than authority. Verify:

- intended externally observable behavior and accepted architectural direction;
- explicit current-slice non-goals and deferred behavior;
- affected users, callers, data, trust boundaries, and semantic concepts;
- compatibility, migration, rollout, lifecycle, and recovery expectations where applicable;
- the highest-risk failure scenarios.

Use changed concepts to locate relevant documentation; do not begin by reading every document in the repository. Resolve meaningful evidence conflicts under the discovery checklist. If unresolved intent prevents a safe conclusion, ask a focused question rather than inferring a defect.

## 3. Inspect CI and check status

Identify which commit was checked and what each required job actually runs. Compare the checked commit and range with the Review Brief's target. Distinguish passed, failed, skipped, allowed-to-fail, and absent checks. State explicitly when an included staged, unstaged, or untracked overlay has no remote CI evidence. Green CI is an input to review, not the conclusion.

## 4. Inspect surrounding implementation

Follow callers, callees, shared contracts, types, schemas, persistence operations, and package boundaries far enough to validate the complete change. Do not review only patch hunks when behavior depends on unchanged code. Use the Review Brief's changed-concept inventory to inspect repository-wide owners and relevant contracts proportionately.

## 5. Perform the module cohesion, public-contract, and policy-ownership audit

For changed files and their semantic neighbors, use size, method count, export concentration, adapter/protocol density, and literal concentration only as signals to inspect. Determine whether a module owns independently evolving responsibilities such as contracts, transport mechanics, authentication, lifecycle, protocol classification, DTO mapping, persistence, validation, logging, retry/effect policy, or route construction. Do not report a large cohesive parser or require arbitrary splitting.

Identify public interfaces/types/ports/constants and their consumers. Confirm that consumers do not need to import a concrete implementation just to use an independently owned contract, but preserve useful colocation where the contract is local to that implementation. Trace status/method/route/failure/effect/retry mappings to determine whether distant helpers are one hidden policy matrix or intentionally separate owners. For protocol-sensitive literals, search repository-wide for existing constants, schemas, OpenAPI/routes, parsers/formatters, enums, and classifiers before recommending reuse or a focused new owner.

If decomposition is warranted, name the independent responsibilities, the drift or audit risk, the semantic boundaries, what remains together, behavior-preserving scope, and tests that protect independently testable policy areas. Do not write “file is too large,” create a constants junk drawer, or require one interface per file.

## 6. Perform the repository-wide duplication and reuse audit

For each changed or newly introduced protocol, business, storage, security, or lifecycle concept, search the repository—not only the diff—for existing helpers, types, schemas, constants, route strings, regexes, serialization formats, error/status codes, storage prefixes, and policy wording. Classify matches as textual, structural, or semantic duplication; identify the authoritative owner and existing primitive; and report only concrete drift, boundary, or reuse risks. Do not turn this step into a blanket DRY rule: preserve local code when semantics or policies intentionally differ.

## 7. Inspect affected tests

Confirm the runner discovers the tests and that assertions establish the intended behavior, failures, edge cases, and side effects. Check whether mocks hide the relevant integration and whether test labels match their real scope. For lifecycle or concurrency paths, apply the semantic checklist's exact-interleaving guidance rather than accepting the presence of the same actors and operations as behavioral coverage.

## 8. Inspect architecture boundaries

Check conceptual responsibilities, ownership, dependency direction, package exports, and transport/application/infrastructure separation according to the project's chosen architecture. For layered, Clean, or Hexagonal designs, inspect imports and contracts across domain/core, application/use cases, ports, transport, infrastructure/adapters, and framework/composition. Confirm that inner layers do not import concrete outer concerns, that application-owned outbound ports describe use-case capabilities, and that infrastructure adapts to those ports. Inspect for storage/transport representations, SDK types, framework APIs, and adapter vocabulary leaking inward; do not flag an abstract persistence port merely because it concerns storage, and do not review by directory name alone.

## 9. Inspect data-safety and security implications

Trace destructive and mutating paths. Consider concurrent writers, conditional updates, rollback, retries, idempotency, validation, authorization, permission changes, path handling, sensitive values, unsafe rendering/execution, and agent prompt/content trust boundaries.

Treat a credible silent-data-loss race or security-boundary failure as blocking even when CI is green.

## 10. Inspect diagnostics and deprecations

Review available compiler, linter, formatter, IDE/editor, framework, generated API/schema, and deprecation diagnostics. Determine whether CI exercises the relevant configuration and type-aware modes. Do not assume a successful build reveals all useful warnings.

When bundling, code generation, framework transformation, packaging, compilation, or runtime adaptation could change the reviewed behavior, verify the important invariant against the generated or runtime artifact in addition to source-level tests where practical. Keep this proportional to risk and repository conventions; artifact testing is not a universal requirement.

## 11. Inspect coverage changes

Review lines, statements, functions, and branches where available. Check source inclusion, exclusions, new uncovered files, risk-sensitive gaps, and threshold changes. Coverage is evidence of execution, not proof of good assertions.

## 12. Perform the manual semantic review

Use [`semantic-review.md`](semantic-review.md). Ask whether the implementation is correct, preserves invariants, handles failures, protects data, keeps architecture coherent, and stays no more complex than necessary. Explicitly trace changed validators, consistency checkers, protocol classifiers, lifecycle code, and state machines for independent invariant families and hidden state/action matrices; do not infer simplicity from shallow nesting, individually simple guards, or green branch coverage.

## 13. Classify and write findings

Use the severity model in [`../SKILL.md`](../SKILL.md). Each finding must include a precise location, consequence, evidence, and bounded remediation direction. Separate blockers from non-blocking notes. Do not report preference as defect or manufacture a finding when approval is warranted.

## 14. Re-review after fixes

Rediscover the same active change through the discovery checklist, recover prior findings from current conversational context, and retain the original baseline when it is still valid. Do not require the user to repeat the target, original brief, or findings. If the repository or target changed unexpectedly, resolve that ambiguity; if the complete diff, governing evidence, scope, or risk materially changed, rebuild the Review Brief.

Inspect both the corrective delta and the current complete target, then rerun or re-check relevant canonical validation. Fixes can introduce regressions, alter contracts, or address only the visible symptom. For lifecycle or concurrency fixes, verify not only that the original scenario now passes but also that state and responsibility moved to the correct lifetime and layer, the fix is not merely another flag in the wrong abstraction, no stale-state or reset regression was introduced, the invariant is owned by the narrowest correct layer, and the tests reproduce the exact original failure interleaving.

For every prior finding, record exactly one status:

| Status | Meaning |
|---|---|
| `fixed` | The cause and relevant regression risk are addressed. |
| `partially fixed` | Some impact remains; explain what and retain appropriate severity. |
| `not fixed` | The issue remains; cite current evidence. |
| `explicitly deferred with acceptable rationale` | The owner accepted a bounded non-blocking risk with a credible reason or follow-up. |
| `withdrawn / not applicable` | New contract or implementation evidence disproves the original finding; cite that evidence and correct the review record. |

## 15. Close corrective review deliberately

Do not say "looks good" only because CI is now green. Approve only when every previous finding has been verified and all remaining issues are genuinely non-blocking. If a blocking finding is deferred without an acceptable safety rationale, keep the request for changes.

## Review-after-green-CI quick check

When every automated check passes, spend review attention on what those checks usually cannot prove:

- the requested behavior is the behavior implemented;
- public contracts, schemas, and runtime behavior agree;
- concurrency and partial failures cannot corrupt or lose data;
- authorization and trust boundaries remain intact;
- architecture, module cohesion, public contract placement, and policy ownership remain coherent;
- protocol-sensitive literals and status/effect/retry mappings have a repository-wide semantic owner where applicable;
- diagnostics omitted by CI have been considered;
- tests execute and assert meaningful behavior rather than mocks or percentages;
- rollout, compatibility, observability, and recovery are adequate.
