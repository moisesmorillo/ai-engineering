# Design feasibility review skill

## Objective

Provide a portable review skill for implementation-facing PRDs, ADRs, specifications, and architecture documents, while keeping `code-review` focused on implementation, tests, executable configuration, and their accompanying documentation.

## Problem and scope

`code-review` audits code-adjacent documentation for semantic quality but has no feasibility gate for design content: it never sums aggregate platform budgets. A design-only change in a downstream repository passed local validation while exceeding the platform's per-invocation subrequest limit. This work adds a design review contract, unambiguous routing between the two skills, a rule for composing their verdicts, and a manual validation scenario. It changes skill instructions and the README only.

## Constraints

- Preserve portable guidance, repository-local authority, and the existing semantic documentation audit for code changes.
- Implementation-facing design documents always take the design route, even when they ship with code. Purely editorial documentation triggers neither skill.
- Do not restate `code-review` policy inside the new skill; reference its discovery checklist, closure vocabulary, and severity scale instead.

## Tasks

- [x] T1 — Add `design-feasibility-review` with activation boundaries, primary-source limit verification, aggregate worst-case budgets, failure/concurrency counterexamples, corrective closure, severity-to-verdict mapping, and an evidence-based verdict.
- [x] T2 — Narrow `code-review` to non-design documentation, define hand-off and both-skills behavior, and route changes in the README with a verdict composition rule.

## Acceptance criteria

- Design-only implementation contracts receive quantified feasibility review, including plan or tier assumptions and aggregate per-operation resource limits.
- Corrective review dispositions every prior finding and recalculates the whole design after a fix.
- Every change maps to exactly one routing row; an implementation-facing design document is never treated as accompanying documentation.
- When both skills run, the more restrictive verdict governs.
- Guidance is model agnostic and does not claim that one model or green CI proves correctness.

## Validation evidence

- No runtime harness applies: these are instruction artifacts and the repository has no CI or test runner. Validation is manual scenario review, frontmatter and relative-link checks, and `git diff --check`.
- Manual scenario: a design performs 10,000 head GETs + 200 LIST calls + 128 lane observations per invocation = 10,328 subrequests to Cloudflare internal services. Limit source: [Workers platform limits, "Subrequests"](https://developers.cloudflare.com/workers/platform/limits/), retrieved 2026-09-28: subrequests to internal services are 1,000 per invocation on the Free plan and 10,000 by default on the Paid plan. The design exceeds both, so the aggregate-budget step returns `not ready` before approval.
- Routing check: a code change with an ADR maps only to the "both skills" row; a code change with a changelog entry maps only to the `code-review` row.

## Rollback boundary

T1 removes the new skill directory. T2 restores the prior `code-review` description and README routing. Neither touches runtime code.
