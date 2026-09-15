# Corrective Re-review and Policy-Matrix Examples

These examples exercise finding closure and structural-candidate persistence. They are calibration cases, not universal architecture or severity rules.

## A — Correctness fixed, structure still open

### Initial review

`src/remote/fetch-transport.ts` was selected as a structural candidate. The review reported:

- **C1 (MAJOR, blocking):** two response paths classify concrete failures incorrectly.
- **S1 (MINOR, non-blocking):** the transport module mixes request/lifecycle mechanics, protocol classification, DTO mapping, and mutation-effect policy. Several helpers collectively implement `status -> failure -> effect certainty -> retry/pause behavior`.

### Corrective change

The correction fixes both response bugs and moves route/header spellings into shared protocol constants. Tests are green. The final module still:

- dispatches requests and owns timeout/abort settlement;
- maps remote DTOs;
- classifies statuses into application failures;
- derives effect certainty and retry/pause behavior in separate helpers; and
- requires action-specific edits in several places when a status is added.

### Expected corrective review

The reviewer reads the complete final module and helper call chain, not only the bug-fix/constants delta. A compact closure record is sufficient:

| Prior finding | Prior status | Disposition | Evidence |
|---|---|---|---|
| C1 — incorrect response classification | MAJOR, blocking | `fixed` | The final branches and regression tests cover both original failures. |
| S1 — mixed transport and distributed effect policy | MINOR, non-blocking | `still open — partially remediated` | Route/header spelling now has one owner, but status/failure/effect/retry policy remains distributed and independently editable. |

The structural-candidate reassessment inventories the remaining mechanics, policy, mapping, lifecycle, contracts, protocol knowledge, and matrix owners. It states that literal ownership improved but the original policy-audit risk did not fully resolve.

**Verdict calibration:** plain `APPROVE` is invalid because an actionable structural concern remains. `APPROVE WITH NOTES` is allowed only when the residual concern is explicitly justified as non-blocking (or intentionally deferred with accepted scope/evidence). If S1 was blocking and remains unresolved, the verdict is `REQUEST CHANGES`. Fixing C1 and extracting constants never closes S1 by implication.

## B — Structural concern genuinely resolved

### Initial review

A large adapter mixed transport orchestration, reusable public contracts, DTO conversion, and status/effect/retry policy. The structural finding identified those independent semantic owners and required focused policy tests.

### Corrective change

The final implementation:

- keeps cohesive request construction, dispatch, bounded-body handling, and immediate transport settlement in the adapter;
- moves reusable contracts to their existing application/port owner;
- moves DTO translation to a boundary mapper;
- gives one typed protocol classifier ownership of action/status/failure/effect/retry rows; and
- adds focused matrix and mapper tests while preserving externally observable behavior.

### Expected corrective review

The candidate remains selected for the corrective pass even though the diff already looks like a refactor. The reviewer inventories the final module and call chain, confirms the new semantic owners and import direction, and verifies that additions no longer require uncoordinated edits across classifiers.

| Prior finding | Disposition | Evidence |
|---|---|---|
| Mixed adapter ownership and hidden policy matrix | `fixed` | Final ownership boundaries match independently evolving concerns; the typed classifier exposes material rows; focused tests exercise policy and mapping independently. |

The adapter may still be large. Closure is justified by cohesion, ownership, auditability, and test boundaries—not line count or the mere existence of new files.

## C — Policy literals without textual duplication

One module uses each of these values exactly once:

```text
PUT + 201 -> success
PUT + 409/412/428 -> refused, effect absent, caller may reconcile
PUT + 429/5xx -> unavailable, effect uncertain, pause before retry
DELETE + 404 -> successful idempotent absence
DELETE + 409 -> refused, effect absent
```

Separate helpers map status to failure, failure to mutation-effect certainty, and action/effect to retry or pause behavior. No repository constant or duplicate literal currently exists.

### Expected review

The reviewer still constructs and assesses the conceptual `method/action + status -> failure -> effect -> retry/pause` matrix. The reviewer checks overlapping branches, impossible or missing rows, action-specific differences, refusal/retry knowledge, independent edit points, and tests for material rows.

A finding, if warranted, concerns the hidden or distributed policy owner and concrete drift/audit risk. It does **not** claim that numbers are inherently magic or demand constants for every status. If the helpers intentionally belong to separate semantic layers with explicit contracts and complete tests, the reviewer may explain that ownership and report no finding.

## D — Ordinary literal

A request builder contains one local `method: "GET"`, and its immediately adjacent response check treats a local `404` as “optional resource absent.” No other action, effect-certainty, retry, compatibility, schema, or cross-module policy depends on either value.

### Expected review

No magic-literal or extraction finding. The values are locally obvious and do not form or duplicate a policy matrix. Do not introduce constants, an enum, a table, or a `switch` merely to hide them.
