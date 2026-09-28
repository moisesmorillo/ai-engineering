---
name: design-feasibility-review
description: "Trigger: implementation-facing PRD, ADR, specification, or architecture review, including corrective re-review of a corrected design. Tests feasibility against verified platform limits with aggregate worst-case budgets, failure and concurrency counterexamples, and evidence-based closure."
license: MIT
---

# Design Feasibility Review

## Activation Contract

Review implementation-facing PRDs, ADRs, specifications, and architecture changes before they are called ready to build or merge. When the change also includes code, run `code-review` as well; the more restrictive verdict governs the merge decision. Skip purely editorial documentation unless explicitly asked.

## Normal invocation

```text
/skill:design-feasibility-review
```

Discover the repository, target, baseline, and range the same way `code-review` does: follow its [target and context discovery checklist](../code-review/checklists/review-discovery.md). Resolve the design document set from the diff, then locate the accepted requirements, platform assumptions, adjacent contracts, and current implementation boundary that govern it. An explicit user target or focus is an override, not a prerequisite.

## Hard Rules

- Follow the target repository's accepted requirements, platform assumptions, and local instructions. Do not turn a bounded milestone into an unrelated redesign.
- Do not equate green CI, model choice, or the author's self-review with design feasibility.
- Disclose whether the reviewer is independent of the author and authoring run. Never label self-review or a second pass in the same run independent.
- Verify time-sensitive platform limits against current primary documentation; cite the exact source, plan or tier, and retrieval date. Mark inaccessible or unknown limits unresolved, never guessed.
- Absence of evidence is not evidence of safety. Do not conclude that a race cannot occur because no test exercises it, that an external effect did not happen because the response was lost, or that a precondition holds because the design does not mention it failing. Each such gap is a finding, not a pass.

## Decision Gates

| Situation | Action |
| --- | --- |
| Target or baseline unclear after discovery | Ask one concise question for only the missing target or baseline; do not guess. |
| Plan or tier unspecified | Do not assume the most permissive tier. Bound every budget by the most restrictive tier the repository could plausibly run on, state that assumption in the report, and record fixing the tier as a follow-up. A budget that passes under that tier caps the verdict at `ready with bounded follow-ups`; a budget that fails under it is `not ready`. |
| Design claims support for several tiers or scales | Evaluate every claimed tier separately. A claimed tier whose budget fails makes the support claim a MAJOR finding, and the verdict is `not ready` while any claimed tier fails. |
| No implementable contract or measurable acceptance criteria | Request the missing contract or criteria before approval. |
| Correction to an earlier finding | Apply step 5: recompute the whole resulting design, not just the changed paragraph. |

## Execution Steps

1. Inspect the complete design change, accepted scope, adjacent contracts, migration path, and current implementation boundary.
2. List invariants, authority and trust boundaries, supported scale, compatibility, rollback, and recovery conditions.
3. For every consequential operation, calculate worst-case **aggregate** calls, subrequests, bytes, time, memory, per-key writes, and storage growth where applicable. Show formulas, inputs, headroom, plan-specific limits, and the behavior when a limit is reached. Count new work introduced by proposed fixes.
4. Challenge ordering, concurrency, retries, partial results, stale clients, interruption, and uncertain effects with concrete counterexamples. Check that tests cover the boundary case and recovery path.
5. On corrective review, recover every prior actionable finding from the current conversation, prior report, or associated review context, and give each exactly one disposition using the [`code-review` closure vocabulary](../code-review/SKILL.md#corrective-finding-closure-and-structural-persistence): `fixed`, `intentionally deferred`, `rejected`, or `still open`, each with evidence. A finding never disappears because the corrected paragraph no longer mentions it. Then repeat steps 2–4 against the final design and recheck every budget and interface the correction touches.

## Severity and Verdict

Use the [`code-review` severity scale](../code-review/SKILL.md#severity-and-prioritization): `BLOCKER`, `MAJOR`, `MINOR`, `NIT`. Assign severity from demonstrated impact and likelihood. An exceeded or unverifiable limit that a budget depends on is at least `MAJOR`; an exceeded limit on a required operation with no defined behavior at the limit is `BLOCKER`.

| Open findings | Verdict |
| --- | --- |
| Any `BLOCKER` or `MAJOR` | `not ready` |
| Only `MINOR` or `NIT` | `ready with bounded follow-ups` |
| None | `ready` |

A corrective review cannot return `ready` or `ready with bounded follow-ups` while a prior `BLOCKER` or `MAJOR` finding is `still open`. When `code-review` also ran, report both verdicts and the governing one.

## Output Contract

Return the target/baseline; reviewer independence or lack of it; scope and assumptions, including any assumed tier; a compact resource-budget table with formulas, plan-specific limits, headroom, and primary-source links with retrieval dates; evidenced findings with severity and exact location; counterexamples and missing tests; prior-finding dispositions when applicable; and the verdict. State every unverified limit and its effect on the verdict.

## References

- [`code-review` discovery checklist](../code-review/checklists/review-discovery.md) — target, baseline, and range resolution.
- [`code-review` corrective closure](../code-review/SKILL.md#corrective-finding-closure-and-structural-persistence) — disposition vocabulary and closure evidence.
- [`code-review` severity scale](../code-review/SKILL.md#severity-and-prioritization) — severity definitions reused here.
