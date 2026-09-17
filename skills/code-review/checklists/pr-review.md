# Change and Pull Request Review Workflow

Follow this order for an initial review of a PR, branch, commit range, working-tree change, or explicit repository-wide audit. Increase depth for high-risk changes; do not skip the final semantic pass because automation is green. In explicit baseline mode, use discovery's source inventory and risk-ranked candidates instead of diff/range steps; no PR is required, and CI evidence must be tied to the baseline actually reviewed.

## 1. Discover the review target and context

Run [`review-discovery.md`](review-discovery.md) before evaluating findings, including for a no-argument `/skill:code-review` invocation. Resolve the repository, target, baseline/range, intentional local overlay, governing evidence, accepted scope, semantic owners, risk profile, and canonical validation into the internal Review Brief. An explicit user target overrides automatic target selection; user focus remains additive to autonomous discovery.

If discovery leaves materially different targets or baselines plausible, ask one concise question and stop before findings. Do not guess. Keep the detailed precedence and evidence-conflict algorithm owned by the discovery checklist rather than duplicating it here.

## 2. Confirm the complete diff and intended change (or baseline scope)

For a change review, read every production, test, configuration, schema, generated, dependency, and documentation file in the Review Brief's complete range, including any intentionally included working-tree overlay. For an explicit baseline review, confirm source roots, exclusions, and risk-ranked candidates, then inspect the current source/contracts rather than demanding a diff. Watch for scope creep, generated or lockfile churn, hidden deletions, changed defaults, and behavior split across commits or files. Do not review only the latest commit of a multi-commit change.

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

## 5. Select and inspect structural maintainability candidates

Before deciding cohesion findings, select candidates. For a PR, inspect materially changed modules and nearby semantic owners with high-signal characteristics: unusual relative size; many methods or exports; imports spanning architectural concerns; protocol-heavy adapter code; transport/mapping/policy/lifecycle/contracts in one module; coordination across several state/effect/evidence dimensions; clusters of distinct helpers; repeated status/header/method/action literals; or broad manager/service/adapter/helper modules. These signals select candidates only. They do not create findings or imply a line, method, export, or complexity threshold.

For every selected candidate, create a concise responsibility inventory: independently evolving responsibilities; mechanics versus policy; architectural owners; concerns that can change or be tested independently; public contracts with separate import/use lifecycles; declaration/helper clusters; and knowledge duplicated elsewhere. Conclude explicitly whether the module is cohesive despite size, mixed but low-risk, or has multiple policy owners/credible audit drift. Do not report a large cohesive parser or require arbitrary splitting.

If the candidate materially owns or coordinates stateful or safety-critical behavior, apply the authoritative [`Stateful structural approval gate`](../SKILL.md#stateful-structural-approval-gate). Reconstruct the repository-specific cross-method transition matrix and its ownership before approving; a broad label, small methods, readable guards, colocated code, dedicated constants, or passing tests are insufficient evidence by themselves.

Apply the authoritative [implementation-local declaration and helper ownership guidance](../SKILL.md#implementation-local-declarations-and-helper-ownership), then inspect exported interfaces/types/ports/constants and their consumers. Confirm that consumers do not need to import a concrete implementation just to use an independently owned contract, but preserve useful colocation where the contract is local to that implementation. Follow status/failure/effect/retry/action call chains rather than inspecting helpers in isolation; construct the conceptual matrix and determine whether it has overlapping branches, impossible or missing combinations, action-specific differences, duplicated refusal/retry knowledge, or several independently editable owners—or whether separate layers intentionally own distinct semantics. For protocol-heavy candidates, inventory methods, statuses, headers, media types, route fragments, action strings, and protocol markers as families, then search repository-wide for constants, schemas, OpenAPI/routes, parsers/formatters, enums, and classifiers. Inspect a family collectively even when each literal occurs only once, all are in one file, and no canonical constant exists: the concern is an implicit policy table, not the absence of constants. Assess candidate documentation under the [documentation completeness audit](semantic-review.md#documentation-completeness-and-quality), including private/internal declarations; this audit also applies outside structural candidates.

If decomposition is warranted, name the independent responsibilities, the drift or audit risk, the semantic boundaries, what remains together, behavior-preserving scope, and tests that protect independently testable policy areas. In the final review, visibly record the candidate assessment and link any resulting finding. Do not write “file is too large,” create a constants junk drawer, require one interface per file, or add boilerplate method comments.

## 6. Audit reuse and documentation

### Repository-wide duplication and reuse audit

For each changed or newly introduced protocol, business, storage, security, or lifecycle concept, search the repository—not only the diff—for existing helpers, types, schemas, constants, route strings, regexes, serialization formats, error/status codes, storage prefixes, and policy wording. Classify matches as textual, structural, or semantic duplication; identify the authoritative owner and existing primitive; and report only concrete drift, boundary, or reuse risks. Do not turn this step into a blanket DRY rule: preserve local code when semantics or policies intentionally differ.

### Documentation completeness and quality audit

Run the [semantic documentation audit](semantic-review.md#documentation-completeness-and-quality) using discovery's convention evidence and applicable language profiles. For TypeScript, inventory named declarations in changed files under the [TSDoc policy](../profiles/typescript.md#documentation); prioritize introduced/materially changed contracts and relevant nearby omissions without unrelated cleanup. For explicit baseline reviews, use the repository-wide inventory/search/sampling path and aggregate repeated debt by pattern/owner. Mere comment presence is not semantic coverage.

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

Rediscover the same active change through the discovery checklist, recover prior findings **and previously selected structural candidates** from current conversational or associated review context, and retain the original baseline when it is still valid. Do not require the user to repeat the target, original brief, findings, or candidate list. If the repository or target changed unexpectedly, resolve that ambiguity; if the complete diff, governing evidence, scope, or risk materially changed, rebuild the Review Brief without discarding the prior closure obligations.

Build a closure ledger containing every prior actionable finding and its prior severity/blocking status. Inspect both the corrective delta and the current complete target, then rerun or re-check relevant canonical validation. The delta explains what changed; the final implementation establishes whether the finding is closed. Fixes can introduce regressions, alter contracts, or address only the visible symptom. Green tests, nearby changes, and fixed correctness bugs do not close a broader structural or ownership finding.

For every prior finding, record exactly one disposition:

| Disposition | Meaning |
|---|---|
| `fixed` | The final implementation addresses the cause and relevant regression risk. |
| `intentionally deferred` | An owner or governing scope accepts a bounded residual risk; cite the accepted scope/rationale or follow-up and evidence that it is non-blocking. |
| `rejected` | New contract, implementation, or reproducer evidence disproves the original finding; cite it and correct the review record. |
| `still open` | The issue remains in the final implementation; cite current evidence and current severity/blocking status. |

Treat partial remediation as `still open — partially remediated`, not as closure. For documentation findings, apply the [corrective documentation closure checks](semantic-review.md#corrective-documentation-closure); added comment blocks alone do not establish semantic remediation. State concisely what improved and what remains. A compact table or list is enough; do not repeat the full prose for findings already fixed or disproved.

For lifecycle or concurrency fixes, verify not only that the original scenario now passes but also that state and responsibility moved to the correct lifetime and layer, the fix is not merely another flag in the wrong abstraction, no stale-state or reset regression was introduced, the invariant is owned by the narrowest correct layer, and the tests reproduce the exact original failure interleaving.

### Reassess every prior structural candidate

Re-read the complete final module and relevant call chains, even when the corrective diff touched only bugs, tests, routes, headers, or constants. Re-run a concise inventory of current responsibilities; mechanics versus policy; public/exported contracts; protocol knowledge and literal families; lifecycle/concurrency ownership; distributed classifiers/decision tables; repository semantic owners; and independently testable concerns. State whether and how the correction reduced the original risk.

Trace any conceptual `status -> response classification -> application failure -> mutation effect certainty -> retry/pause behavior` pipeline end to end. Assess overlapping, impossible, and missing combinations; action-specific differences; duplicated refusal/retry knowledge; independent edit points; and whether a new status/action requires coordinated changes across helpers. Constants can centralize spelling while leaving policy ownership distributed, so do not infer that extracting them fixed the matrix. Prefer an explicit typed table, canonical classifier, exhaustive dispatch, or equivalent only when it materially improves auditability; do not require a split because a module remains large or replace every `if` with `switch`.

For every materially stateful carried candidate, apply the [`Stateful structural approval gate`](../SKILL.md#stateful-structural-approval-gate) to the final resulting code: rediscover the matrix, compare responsibility and transition-policy ownership before versus after, and explicitly disposition the structural concern. Functional bug fixes do not establish structural closure.

The reassessment may conclude that no structural finding remains, but it must explain the final semantic owners, boundaries, testability, and matrix auditability that justify closure.

## 15. Close corrective review deliberately

Emit the compact finding-resolution ledger and the reassessment for every prior structural candidate. Do not say "looks good" only because CI is now green.

- Return `REQUEST CHANGES` while any prior blocking finding remains unresolved. A deferral changes this only when governing policy or an authorized owner explicitly accepts bounded scope and evidence shows the residual no longer blocks approval.
- Use `APPROVE WITH NOTES` only when every remaining actionable concern is non-blocking and explicitly, credibly dispositioned.
- Use plain `APPROVE` only when no actionable finding remains and every applicable stateful structural candidate passed the approval gate on the final code.

Fixing functional defects within a structural candidate does not automatically resolve its structural finding. Conversely, structural concerns are not automatically blocking: calibrate severity and disposition from demonstrated residual risk.

## Review-after-green-CI quick check

When every automated check passes, spend review attention on what those checks usually cannot prove:

- the requested behavior is the behavior implemented;
- public contracts, schemas, and runtime behavior agree;
- concurrency and partial failures cannot corrupt or lose data;
- authorization and trust boundaries remain intact;
- architecture, module cohesion, public contract placement, and policy ownership remain coherent;
- protocol-sensitive literals and status/effect/retry mappings have a repository-wide semantic owner where applicable;
- documentation is complete and useful under the discovered convention, including private/internal contracts; comment presence alone does not prove this;
- diagnostics omitted by CI have been considered;
- tests execute and assert meaningful behavior rather than mocks or percentages;
- rollout, compatibility, observability, and recovery are adequate.
