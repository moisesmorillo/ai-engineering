---
name: design-feasibility-review
description: "Trigger: implementation-facing PRD, ADR, specification, or architecture review. Test feasibility, worst-case limits, failures, and corrective closure."
license: MIT
metadata:
  author: moisesmorillo
  version: "1.0"
---

# Design Feasibility Review

## Activation Contract

Review implementation-facing PRDs, ADRs, specifications, and architecture changes before they are called ready to build or merge. Use `code-review` as well when code changes. Skip purely editorial documentation unless explicitly asked.

## Hard Rules

- Follow the target repository's accepted requirements, platform assumptions, and local instructions. Do not turn a bounded milestone into an unrelated redesign.
- Do not equate green CI, model choice, or the author's self-review with design feasibility.
- Disclose whether the reviewer is independent of the author and authoring run. Never label self-review or a second pass in the same run independent.
- Verify time-sensitive platform limits against current primary documentation; cite the exact source and plan or tier. Mark inaccessible or unknown limits unresolved, never guessed.
- Incomplete evidence, uncertain external effects, or failed safety preconditions cannot establish safety-critical absence, authority, or success.

## Decision Gates

| Situation | Action |
| --- | --- |
| Target or baseline unclear | Ask only for the missing target. |
| Unsupported or unspecified plan/scale | State the assumption and evaluate every claimed supported tier; block unsupported claims. |
| No implementable contract or measurable acceptance criteria | Request the missing contract or criteria before approval. |
| Correction to an earlier finding | Recompute the whole resulting design, not just the changed paragraph. |

## Execution Steps

1. Inspect the complete design change, accepted scope, adjacent contracts, migration path, and current implementation boundary.
2. List invariants, authority and trust boundaries, supported scale, compatibility, rollback, and recovery conditions.
3. For every consequential operation, calculate worst-case **aggregate** calls, subrequests, bytes, time, memory, per-key writes, and storage growth where applicable. Show formulas, inputs, headroom, plan-specific limits, and the behavior when a limit is reached. Count new work introduced by proposed fixes.
4. Challenge ordering, concurrency, retries, partial results, stale clients, interruption, and uncertain effects with concrete counterexamples. Check that tests cover the boundary case and recovery path.
5. On corrective review, disposition each prior finding with evidence, then repeat steps 2–4 against the final design. Recheck all affected budgets and interfaces.

## Output Contract

Return the target/baseline; reviewer independence or lack of it; scope and assumptions; a compact resource-budget table with formulas and primary-source links; evidenced findings with severity and exact location; counterexamples and missing tests; prior-finding dispositions when applicable; and a verdict (`ready`, `ready with bounded follow-ups`, or `not ready`). State every unverified limit and its effect on the verdict.

## References

No local supporting references.
