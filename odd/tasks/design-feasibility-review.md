# Design feasibility review skill

## Objective

Provide a portable review skill for implementation-facing PRDs, ADRs, specifications, and architecture documents, while keeping `code-review` focused on code changes and their accompanying tests and documentation.

## Problem and scope

The existing review skill covers semantic code and code-adjacent documentation but has no dedicated feasibility gate for design-only changes. PR #91 in `obsidian-ai-bridge` exposed a missing aggregate resource-budget check despite local validation. This feature adds a design review contract, explicit routing, and a small manual validation exercise. It does not change other repositories or configure OpenCode models.

## Authorization and constraints

The user authorized a PR in `moisesmorillo/ai-engineering`. Preserve portable guidance, repository-local authority, and the existing semantic documentation audit for code PRs. Purely editorial documentation does not trigger feasibility review. Mixed PRs use both skills. Remote push and PR creation await explicit credential/session authorization.

## Route and delivery

Delegated direct. Mapping trigger: the existing skill, checklists, README, and local style guide need inspection. Writer trigger: the new skill and routing documentation require multiple nontrivial edits. Preparation mapping was delegated before source edits. Forecast: under 400 authored changed lines; single PR. TDD: no repository configuration or test runner; use manual scenario review, frontmatter/link checks, and diff hygiene.

## Tasks

- [x] T1 — Add `design-feasibility-review` with activation boundaries, official-source verification, quantitative worst-case limits, failure/concurrency counterexamples, corrective review, and an evidence-based verdict. Check its structure and apply it to the M7 design scenario. Commit the work unit.
- [x] T2 — Clarify `code-review` code scope, route design and mixed PRs in README, and preserve code-adjacent documentation review. Check routing consistency and repository links. Commit the work unit.

## Acceptance criteria

- Design-only implementation contracts receive quantified feasibility review, including platform plan assumptions and aggregate per-operation resource limits.
- Corrective review recalculates the whole design after a fix, rather than checking only the original finding.
- Code-only and mixed PR routing is unambiguous; editorial docs do not require a design review.
- Guidance is model agnostic and does not claim that one model or green CI proves correctness.

## Progress and next step

Branch `feat/design-feasibility-review` created from clean main. Mapping and bounded writing completed. T1 committed as `08b80b5` (`feat(skills): add design feasibility review`); T2 committed as `7303f67` (`docs(skills): route code and design reviews`). Structure/frontmatter checks and `git diff --check` passed. Manual M7 scenario: 10,000 head GETs + 200 LIST calls + 128 lane observations = 10,328 subrequests, above Workers Free internal-service 1,000 and Paid default 10,000; the new skill's aggregate-budget step would flag this before approval. T2 routing and relative README links passed a focused scripted check; code-adjacent documentation remains in `code-review`, design-only changes use the new skill, mixed changes use both, and editorial-only changes use neither by default. Runtime harness: N/A, as these are instruction artifacts. Rollback boundary: T1 removes the new skill; T2 restores the prior README and reviewer routing without touching runtime code. Native RDD assessment was unavailable because `gentle-ai` is not installed. Engram mirror `odd/design-feasibility-review/tasks` pending because no Engram tool is exposed in this runtime. Next: publish PR after remote credential authorization.
