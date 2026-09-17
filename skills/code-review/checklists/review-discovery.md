# Review Target and Context Discovery

This is the authoritative discovery algorithm for the code-review skill. Run it before evaluating or reporting findings, including when the user invokes only `/skill:code-review`. Derive the review target and a concise internal Review Brief from repository evidence; do not require the user to prepare a review prompt when the active development environment is sufficient.

Discovery must be proactive, evidence-based, and proportionate. Record the evidence supporting each derived constraint. Do not infer architectural intent from filenames alone, silently choose a convenient range, or read every repository document before inspecting the change.

## 1. Resolve the repository and review target

Resolve the repository root from an explicit user override first, then from the current directory, active worktree, and session/tool context. Do not ask for a repository URL when the local environment already identifies the repository.

Choose the review target using this precedence:

1. An explicit target, range, PR, file set, or repository-wide audit supplied by the user.
2. A verified active/current PR associated with the current branch or session.
3. The current branch's complete change against its appropriate base.
4. Staged and unstaged working-tree changes, plus relevant untracked files, when no meaningful PR or branch diff exists.
5. An explicitly requested repository-wide or baseline review when no change target is intended.
6. If no meaningful target can be inferred safely, ask one concise question for only the missing repository, target, base, or working-tree choice. Produce no findings until it is resolved.

Use local Git state, branch and worktree metadata, remotes, session context, and hosting metadata when available. Hosted PR discovery is useful, not mandatory. Never require a repository URL, PR number, base SHA, milestone, or slice number that these sources already establish. Never silently review the latest commit, an arbitrary historical range, or the entire repository.

Treat PR title/body, issue text, and review descriptions as useful intent evidence but not authority. Verify them against the complete diff, repository contracts, current design sources, and tests. A misleading or incomplete description must not shrink the review.

In change mode, when a hosted PR is associated with the current branch/session, compare its verified head SHA with local `HEAD` and record both. Use the hosted head when they match. If local `HEAD` is a descendant of the hosted head on the same change branch, review through local `HEAD` as an unpublished committed extension of the PR and disclose which commits lack hosted CI evidence. If the hosted head is a descendant of local `HEAD`, review the hosted PR head after making its objects/diff available. If the heads diverge, their relationship cannot be refreshed, or the local commits may belong to a different change, ask one concise question when the alternatives would materially change the review. Do not silently omit either side or conflate an unpublished commit with an uncommitted working-tree overlay.

Keep committed and local changes distinct. A PR/branch review normally covers its committed semantic range; include an uncommitted overlay only when the user or current corrective-work context makes it part of the target, and state that inclusion. Do not silently include unrelated dirty files or imply that remote CI validated them. When the working tree is the only target, use the staged/unstaged changes and relevant untracked files and do not ask for a PR number.

## 2. Establish the correct baseline and range

**Explicit repository-wide mode:** “review the whole repo,” “repo-wide review,” and “baseline review” select the current source baseline (or an explicitly named revision), even if no PR or diff exists. Record the revision, roots, exclusions, and any intentionally included working-tree overlay. Do not ask for a PR/base or manufacture a comparison range. Skip change-range selection below; substitute a source/concept inventory for the diff steps that follow, then build risk-ranked candidate sets and run the [documentation completeness audit](semantic-review.md#documentation-completeness-and-quality) alongside the existing semantic audits.

**Change mode:** use the strongest available baseline evidence in this order:

1. The user's explicit range or baseline.
2. The verified PR base branch and its merge base with the reviewed head.
3. A repository-supported integration/base branch derived from branch configuration, documented workflow, upstream relationships, or other current repository evidence.
4. The repository's verified default branch and its merge base with the reviewed head.
5. `HEAD` only as the baseline for a working-tree-only review.

A feature branch's same-name tracking ref is not automatically its comparison base. Do not mechanically assume `main` or `master`. Prefer current remote/hosting evidence; refresh refs when safe and practical. If a relevant ref could be stale and cannot be refreshed, disclose that limitation rather than presenting the range as certain.

Review the complete semantic change, normally equivalent to:

```text
merge-base(base, head)..head
```

or the hosting provider's equivalent complete PR diff. Include every commit in a stacked or multi-commit PR; do not reduce it to `HEAD^..HEAD`. Record the base branch/ref, merge-base commit, target head, included local overlay, exclusions, and why that range was selected.

If multiple bases or targets remain plausible and would produce materially different reviews, ask one concise clarifying question rather than guessing.

## 3. Inspect the diff, then discover governing context

Use this order:

```text
discover target and baseline
        ↓
inspect the complete diff (or inventory the explicit source baseline)
        ↓
identify changed concepts (or baseline capabilities/contracts)
        ↓
discover relevant repository rules, specs, plans, ADRs, and tests
        ↓
identify semantic owners and risk
        ↓
select structural candidates and inventory responsibilities
        ↓
build the internal Review Brief
        ↓
perform detailed review
        ↓
run or inspect canonical validation
        ↓
perform the final semantic pass
```

After reading the complete diff (or inventorying the explicit baseline), identify semantic concepts and architectural layers/capabilities. Use those concepts, relevant paths, imports, types, functions, protocols, and nearby tests to search selectively for applicable:

- `AGENTS.md`, README, and contribution instructions;
- architecture and current-state documentation;
- roadmap, milestone, feature, migration, and rollout plans;
- specifications, ADRs, accepted design decisions, and protocol documentation;
- package/module documentation and established neighboring implementation;
- CI configuration, task definitions, and tests that demonstrate contracts.

Discover languages, source roots, and documentation conventions from package/build configuration, instructions, doc tooling, and representative source. Load the [TypeScript profile](../profiles/typescript.md) whenever TypeScript is in scope, without waiting for the user to request TSDoc review. Repeated undocumented internals alone are not evidence of an intentional exception to its default.

Read applicable repository-root instructions and any more specific instructions governing reviewed paths. Do not read every document blindly. Every dynamically derived invariant, non-goal, architectural direction, and validation expectation must be traceable to repository or user evidence.

## 4. Select structural maintainability candidates

After the initial diff/concept inventory and before detailed review, deliberately select structural candidates. For a PR, apply this proportionally to materially changed modules and their direct semantic owners. For an explicit repository-wide or baseline review, actively identify a small, risk-ranked shortlist of high-signal source files/modules for deeper inspection; do not treat every large file as a candidate or turn the exercise into a full redesign.

Signals include unusual size relative to the repository, many methods or exports, imports spanning architectural concerns, protocol-heavy adapter code, mixing transport/mapping/policy/lifecycle/contracts, coordination across several state/effect/evidence dimensions, distinct helper or declaration clusters, repeated status/header/method/action literals, and broad manager/service/adapter/helper modules. Signals choose what to inspect only. They are never a finding and create no line, method, export, or complexity threshold.

For each selected candidate, record a concise responsibility inventory in the Review Brief before evaluating findings. In corrective mode, carry forward every candidate selected by the prior review even when it is absent from or only lightly touched by the corrective delta; the final implementation, not recent line churn, determines whether its structural risk remains.

- independently evolving responsibilities, distinguishing mechanics from policy;
- architectural owner(s), concerns that can change independently, and concerns that require different tests;
- implementation-local declaration and helper clusters, classified by semantic responsibility rather than count;
- exported contracts/constants with an independent import or use lifecycle;
- status/failure/effect/retry/action call chains that may form one distributed policy matrix;
- protocol literal families—methods, statuses, headers, media types, route fragments, action strings, and protocol markers—and their repository-wide owners; and
- documentation completeness/quality for named semantic declarations, including private/internal contracts, under the applicable language profile and [documentation audit](semantic-review.md#documentation-completeness-and-quality).

Determine whether the candidate materially owns or coordinates stateful or safety-critical behavior. If it does, the Review Brief must capture the repository-specific state dimensions, transition-causing events/results, decision inputs such as budgets/admission/effect/evidence, outcomes, implementation locations, authoritative policy owner, change coupling, and matrix-focused test evidence required by the [`Stateful structural approval gate`](../SKILL.md#stateful-structural-approval-gate).

Conclude each inventory as cohesive despite size, mixed but low-risk, or multiple policy owners/credible audit drift. The final review must visibly state that assessment for selected candidates, including the outcomes for cohesion, contract placement, decision-table ownership, protocol-literal ownership, the stateful gate when applicable, and documentation; it need not manufacture a separate finding for each dimension.

## 5. Build the internal Review Brief

Build a concise working brief before evaluating findings. It is primarily an internal artifact, not a required preliminary response. Include:

### Target and mode

- repository root and relevant worktree;
- initial, corrective, or repository-wide review mode;
- for corrective mode, a closure ledger of every prior actionable finding (identifier/title, prior severity, and prior blocking status) and the carried-forward structural-candidate set;
- branch/PR/range, base, merge base, local and hosted heads when applicable, and selection rationale;
- included committed changes and intentional working-tree overlay;
- explicit exclusions and unresolved target limitations.

### Change intent

- what the change is trying to implement or fix;
- whether it is a feature slice, bug fix, refactor, migration, security change, or another change type;
- affected capabilities, architectural layers, users, callers, data, and trust boundaries.

### Governing sources

- applicable repository instructions, contracts, current ADRs/design decisions, milestone/spec/plan, architecture/current-state docs, tests, and implementation evidence;
- source locations and meaningful conflicts or uncertainty;
- PR/issue descriptions as corroborating rather than controlling evidence.

### Documentation context

- languages and applicable profiles, including TypeScript when present;
- repository documentation convention, its evidence, and any intentional alternative/exemptions;
- named declarations added/materially changed, including internal methods/helpers, types, schemas, and domain constants;
- whether the PR introduces persisted, protocol, state-machine, or other high-risk contracts and where their documentation is owned;
- for baseline mode, production roots/exclusions, declaration inventory, risk-ranked documentation candidates (not just structural candidates), and planned search/sampling coverage.

Use the [semantic documentation audit](semantic-review.md#documentation-completeness-and-quality) and language profile for detailed rules; keep this brief internal.

### Accepted behavior and scope boundaries

- required behavior and acceptance criteria;
- failure and compatibility semantics;
- correctness, security, privacy, data-safety, concurrency, lifecycle, recovery, and operability invariants that apply;
- behavior explicitly deferred or excluded by the current slice.

Respect phased delivery. If current evidence separates transport, orchestration, host wiring, persistence, or network integration into different milestones, review the active slice against its own accepted boundary. Do not report a later slice as missing functionality unless the current change violates a present contract.

### Semantic owners

For changed concepts, identify repository-wide owners and competing implementations for relevant:

- identifiers and canonical formats;
- schemas, parsers, formatters, and validation;
- route and protocol policy;
- state machines and lifecycle rules;
- storage representations and migration policy;
- business policy, authorization, and authentication;
- retry, idempotency, CAS, and network-effect certainty;
- configuration;
- protocol-sensitive literal owners (constants, schemas/OpenAPI, routes, parsers/formatters, enums, and classifiers); and
- exported public contracts and their architectural/import owners.

Use this map to drive the existing repository-wide semantic duplication and reuse audit and the module-cohesion audit. Search repository-wide where ownership is relevant without escalating the review into an unrelated codebase redesign.

### Risk profile

Increase scrutiny based on evidence when the change touches destructive operations, persistence, concurrency/CAS, authentication/authorization, secrets, protocol parsing, migrations, storage formats, recovery, lifecycle/state machines, retries/idempotency, network-effect certainty, or cross-device/process behavior. Also add structural maintainability risk when the diff introduces a very large file, a high concentration of semantic responsibilities, a protocol-heavy adapter, many literals tied to external contracts, or a public contract surface. These are investigation signals, not metric-based findings. Derive review dimensions from the diff and governing evidence; the user should not need to prompt separately for file cohesion, protocol literal ownership, effect certainty, architecture boundaries, protocol acknowledgement validation, ADR compliance, lifecycle races, semantic reuse, or TSDoc completeness/quality when they are inferable.

User-provided focus is additive. It does not replace autonomous discovery. Apply checklist sections proportionately; do not assign severity from a fixed checklist regardless of the actual risk.

### Validation expectations

Identify repository-native validation through package scripts, task runners such as mise, Makefiles, CI workflows, contribution docs, and project-specific tooling. Prefer the repository's canonical command and relevant quality gates. Do not invent generic `npm test`, `make test`, or equivalent commands when another source of truth exists. Record what can be run, what CI ran and for which commit, what was only inspected, and any gaps.

## 6. Resolve evidence conflicts

Use this default precedence while accounting for explicit repository authority and recency:

1. explicit current user requirements;
2. current executable and tested contracts;
3. accepted current ADRs or design decisions;
4. current milestone or feature specifications;
5. architecture and current-state documentation;
6. historical planning/documentation;
7. reviewer defaults.

Do not blindly make a test authoritative when it is stale, encodes a known bug, or conflicts with a stronger accepted contract. Do not select whichever source supports a convenient finding. Identify material conflicts, determine whether recency/status/corroboration resolves them, and report documentation or contract drift when it matters. Ask a focused question only if unresolved ambiguity prevents a safe review conclusion.

## 7. Corrective and repository-wide reviews

For a corrective baseline audit, retain the explicit repository-wide scope, record the new reviewed revision/overlay, and apply the same finding-closure obligations without requiring a PR/diff. For corrective change reviews, use the PR/range rediscovery below.

On corrective re-review, automatically rediscover the same active PR/change, retain the original baseline when still valid, and recover prior findings, prior severity/blocking status, focus, and every previously selected structural candidate from current conversational or associated review context. Do not require the user to paste them again. If complete prior review evidence is genuinely unavailable, state the gap and recover what can be established from hosted review comments or repository artifacts rather than silently treating the corrective pass as finding-free.

Verify every prior actionable finding individually against the final resulting implementation, not only the corrective diff. Apply the [documentation closure checks](semantic-review.md#corrective-documentation-closure) to prior documentation findings. Assign exactly one disposition—`fixed`, `intentionally deferred`, `rejected`, or `still open`—with evidence; record partial remediation as `still open — partially remediated`. Preserve the closure ledger even when the Review Brief is rebuilt because the diff, governing evidence, scope, or risk materially changed. Rejections require new evidence disproving the original finding; deferrals require accepted scope/rationale and evidence that residual risk is bounded and non-blocking.

Carry prior structural candidates into the new brief and re-run their concise responsibility inventory over the final code and relevant call chains: current responsibilities, mechanics versus policy, public/exported contracts, protocol knowledge, lifecycle/concurrency ownership, distributed classifiers/decision tables, semantic owners, independently testable concerns, and whether the correction reduced the original risk. For each materially stateful candidate, reconstruct the final cross-method policy/state matrix, compare its responsibility and transition ownership before versus after, and apply the [`Stateful structural approval gate`](../SKILL.md#stateful-structural-approval-gate) anew. A file need not be split because it remains large, but a structural concern cannot disappear merely because correctness bugs or nearby constants were fixed. If repository/target state switched unexpectedly, resolve that ambiguity before issuing findings.

Normal invocation reviews the active change, not every line of the repository. An explicit repository-wide request selects baseline mode regardless of whether a change target exists. Build the baseline brief and risk-ranked candidate sets, audit documentation completeness across production source, and aggregate repeated debt by pattern/owner with concrete file examples and search evidence. Disclose sampling/coverage limits; do not flood the report with per-symbol omissions. Repository-wide searches for semantic ownership and reuse remain required where relevant to changed concepts, but they do not authorize a total redesign.

## 8. User-visible context

Do not force a long discovery report before the review. Normal use should remain one command producing one review. When useful, include only a short context summary in the final report: target/range, governing spec or ADR, intentional local overlay, and principal risks. Do not dump the internal search process or Review Brief checklist, but visibly include concise assessments for structural candidates selected now or carried from a prior review so the reviewer does not silently skip cohesion, contract placement, distributed policy, literal ownership, or documentation. In corrective mode, also emit the compact finding-resolution ledger; resolved findings need no verbose repetition.
