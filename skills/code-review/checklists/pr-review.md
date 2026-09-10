# Pull Request Review Workflow

Follow this order for an initial review. Increase depth for high-risk changes; do not skip the final semantic pass because automation is green.

## 1. Read repository instructions first

Locate and read applicable `AGENTS.md`, contribution instructions, architecture docs, ADRs, API contracts, CI configuration, and conventions in the changed area. Record which rules are authoritative. Do not substitute this skill's defaults for an intentional local design.

## 2. Understand the intended change

Read the PR description, issue, acceptance criteria, and migration or rollout notes. Identify:

- intended externally observable behavior;
- explicit non-goals;
- affected users, callers, data, and boundaries;
- compatibility and rollout expectations;
- the highest-risk failure scenario.

If intent is unclear, ask focused questions rather than inferring a defect.

## 3. Inspect CI and check status

Identify which commit was checked and what each required job actually runs. Distinguish passed, failed, skipped, allowed-to-fail, and absent checks. Green CI is an input to review, not the conclusion.

## 4. Inspect the actual diff

Read every changed production, test, configuration, schema, generated, dependency, and documentation file. Watch for scope creep, generated or lockfile churn, hidden deletions, changed defaults, and behavior split across commits or files.

## 5. Inspect surrounding implementation

Follow callers, callees, shared contracts, types, schemas, persistence operations, and package boundaries far enough to validate the change. Do not review only the patch hunk when behavior depends on unchanged code.

## 6. Inspect affected tests

Confirm the runner discovers the tests and that assertions establish the intended behavior, failures, edge cases, and side effects. Check whether mocks hide the relevant integration and whether test labels match their real scope.

## 7. Inspect architecture boundaries

Check conceptual responsibilities, ownership, dependency direction, package exports, and transport/application/infrastructure separation according to the project's chosen architecture. Look for direct infrastructure use in handlers and private cross-package imports.

## 8. Inspect data-safety and security implications

Trace destructive and mutating paths. Consider concurrent writers, conditional updates, rollback, retries, idempotency, validation, authorization, permission changes, path handling, sensitive values, unsafe rendering/execution, and agent prompt/content trust boundaries.

Treat a credible silent-data-loss race or security-boundary failure as blocking even when CI is green.

## 9. Inspect diagnostics and deprecations

Review available compiler, linter, formatter, IDE/editor, framework, generated API/schema, and deprecation diagnostics. Determine whether CI exercises the relevant configuration and type-aware modes. Do not assume a successful build reveals all useful warnings.

## 10. Inspect coverage changes

Review lines, statements, functions, and branches where available. Check source inclusion, exclusions, new uncovered files, risk-sensitive gaps, and threshold changes. Coverage is evidence of execution, not proof of good assertions.

## 11. Perform the manual semantic review

Use [`semantic-review.md`](semantic-review.md). Ask whether the implementation is correct, preserves invariants, handles failures, protects data, keeps architecture coherent, and stays no more complex than necessary.

## 12. Classify and write findings

Use the severity model in [`../SKILL.md`](../SKILL.md). Each finding must include a precise location, consequence, evidence, and bounded remediation direction. Separate blockers from non-blocking notes. Do not report preference as defect or manufacture a finding when approval is warranted.

## 13. Re-review after fixes

Inspect the new diff and rerun or re-check the relevant validation. Fixes can introduce regressions, alter contracts, or address only the visible symptom.

For every prior finding, record exactly one status:

| Status | Meaning |
|---|---|
| `fixed` | The cause and relevant regression risk are addressed. |
| `partially fixed` | Some impact remains; explain what and retain appropriate severity. |
| `not fixed` | The issue remains; cite current evidence. |
| `explicitly deferred with acceptable rationale` | The owner accepted a bounded non-blocking risk with a credible reason or follow-up. |
| `withdrawn / not applicable` | New contract or implementation evidence disproves the original finding; cite that evidence and correct the review record. |

## 14. Close corrective review deliberately

Do not say "looks good" only because CI is now green. Approve only when every previous finding has been verified and all remaining issues are genuinely non-blocking. If a blocking finding is deferred without an acceptable safety rationale, keep the request for changes.

## Review-after-green-CI quick check

When every automated check passes, spend review attention on what those checks usually cannot prove:

- the requested behavior is the behavior implemented;
- public contracts, schemas, and runtime behavior agree;
- concurrency and partial failures cannot corrupt or lose data;
- authorization and trust boundaries remain intact;
- architecture and ownership remain coherent;
- diagnostics omitted by CI have been considered;
- tests execute and assert meaningful behavior rather than mocks or percentages;
- rollout, compatibility, observability, and recovery are adequate.
