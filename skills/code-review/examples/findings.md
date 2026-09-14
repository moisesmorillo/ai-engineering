# Examples of Review Findings

These examples demonstrate evidence and impact, not universal project rules. Locations and names are illustrative.

## 1. Bare Hono handler generic weakens binding types

**Severity:** MAJOR

**Bad finding wording**

> Add more Hono types here.

**Strong finding wording**

> **[MAJOR] A bare `Handler` annotation erases the app's Worker binding types**
> **Location:** `src/http/handlers/upload.ts:8`
> **Why it matters:** The parent app is correctly declared as `new Hono<WorkerEnv>()`, but the exported handler is annotated only as `Handler`. In the reviewed Hono version, `Handler` defaults its environment generic to `any`, so `c.env.OBJECT` passes typechecking even though the declared binding is `OBJECTS`; registering that already-widened handler on the typed app does not restore the lost check inside it.
> **Evidence:** Hono declares `Handler<E extends Env = any, ...>`, and this file uses `const upload: Handler = ...`. By contrast, the `Hono` class itself defaults `E` to `BlankEnv`, so `new Hono()` is not the source of broad binding access.
> **Recommended direction:** Parameterize the standalone handler as `Handler<WorkerEnv>`, preserve `E` in any shared composition helper, or let a typed route infer the inline handler context.

**Why the strong version is better:** It identifies the specific broad generic default and composition point, distinguishes it from `Hono`'s `BlankEnv` default, and shows the concrete typo that escapes checking.

## 2. Handler depends directly on infrastructure

**Severity:** MAJOR

**Bad finding wording**

> This violates Clean Architecture. Add another layer.

**Strong finding wording**

> **[MAJOR] The HTTP handler now owns persistence and retry policy**
> **Location:** `src/http/routes/documents.ts:48-91`
> **Why it matters:** The handler constructs `S3DocumentRepository`, decides retryability, and performs the write. That duplicates policy already owned by `SaveDocument` and makes the same operation behave differently for HTTP and queue callers.
> **Evidence:** `src/application/save-document.ts` contains the validation and conditional-write contract, but this route bypasses it and calls the adapter directly.
> **Recommended direction:** Keep request/response mapping in the route and invoke the existing application operation through its repository port.

**Why the strong version is better:** It ties the boundary concern to duplicated behavior and an existing project design rather than imposing an architecture slogan.

## 3. Deprecated API still compiles

**Severity:** MINOR (raise if removal or behavior risk is imminent)

**Bad finding wording**

> Don't use deprecated functions.

**Strong finding wording**

> **[MINOR] New code adopts the runtime's deprecated `legacyEncode` API**
> **Location:** `src/codec/token.ts:27`
> **Why it matters:** The call still compiles, but the installed runtime marks it for removal and its replacement uses the project's required URL-safe behavior. Extending usage increases migration cost and can preserve the wrong encoding semantics.
> **Evidence:** The deprecation diagnostic on this call points to `encodeBase64Url`, already used in `src/codec/session.ts`.
> **Recommended direction:** Use the supported API and add the boundary-case assertion used by the existing codec tests.

**Why the strong version is better:** It cites the actual diagnostic, compatibility consequence, and repository precedent.

## 4. Duplicated protocol/header/status literal

**Severity:** MAJOR

**Bad finding wording**

> Move these strings to constants.

**Strong finding wording**

> **[MAJOR] The upload response defines a second success-status contract**
> **Location:** `src/http/upload.ts:73`
> **Why it matters:** The handler returns `"x-upload-state": "complete"` with status `200`, while the shared protocol and OpenAPI response define `"completed"` with status `201`. Clients generated from the schema will misclassify a successful runtime response.
> **Evidence:** Compare `src/protocol/upload.ts:8-16` and `openapi/uploads.ts:42-58`.
> **Recommended direction:** Derive the handler response from the authoritative upload protocol definition and cover the runtime/schema agreement in a boundary test.

**Why the strong version is better:** It distinguishes semantic drift from harmless repeated strings and describes the client-visible failure.

## 5. `while (true)` hides explicit termination

**Severity:** MAJOR

**Bad finding wording**

> Infinite loops are bad; rewrite this functionally.

**Strong finding wording**

> **[MAJOR] The pagination failure path retries forever with the same cursor**
> **Location:** `src/search/read-all.ts:31-54`
> **Why it matters:** When `fetchPage` returns the handled error, the branch continues without changing `nextCursor` or decrementing a retry budget. The request can therefore repeat the same call indefinitely, consuming resources and never returning a result or terminal error.
> **Evidence:** The optional cursor is the loop's only progress state, and this branch changes neither that state nor any bounded retry state before the next `while (true)` iteration.
> **Recommended direction:** Return a typed terminal error or apply a bounded retry policy, then express normal continuation in terms of cursor presence so termination is visible.

**Why the strong version is better:** It does not reject all unconditional loops or prescribe an FP library; it demonstrates the current failure path and ties the severity to its operational effect.

## 6. Conditional chain should be a discriminated state

**Severity:** MAJOR

**Bad finding wording**

> Too many if statements. Use a switch.

**Strong finding wording**

> **[MAJOR] Independent booleans permit contradictory payment outcomes**
> **Location:** `src/payments/resolve.ts:40-88`
> **Why it matters:** `authorized`, `captured`, `refunded`, and `failed` are checked in a long priority-ordered chain, but the type permits combinations such as `{ captured: true, failed: true }`. The chosen branch then depends on condition order rather than a valid domain state.
> **Evidence:** `PaymentFlags` declares four independent booleans and the new refund branch precedes the failure branch.
> **Recommended direction:** Model the closed outcomes as a discriminated union and handle each state explicitly; retain ordinary guards for independent preconditions.

**Why the strong version is better:** The issue is invalid representable state, not the visual presence of `if` statements.

## 7. Closed-state handling is not exhaustive

**Severity:** MAJOR

**Bad finding wording**

> Add a default case to this switch.

**Strong finding wording**

> **[MAJOR] The new `paused` job state silently receives the `pending` behavior**
> **Location:** `src/jobs/next-action.ts:19-36`
> **Why it matters:** `JobState` is closed and now includes `paused`, but this switch falls through to a post-switch `return "enqueue"`. Paused jobs will therefore be restarted instead of remaining idle.
> **Evidence:** The switch handles `running`, `complete`, and `failed`; `paused` has no branch, and the compiler is not required to prove exhaustiveness here.
> **Recommended direction:** Return from an exhaustive closed-state match and make `paused` behavior explicit.

**Why the strong version is better:** It identifies the missing state and externally observable consequence rather than suggesting a throw-only `default` that could hide future omissions.

## 8. Compile-time exhaustiveness with an artificial runtime default

**Severity:** MINOR

**Bad finding wording**

> This default is ugly and weakens exhaustiveness.

**Strong finding wording**

> **[MINOR] The valid `never` check is carried by an unnecessary runtime default**
> **Location:** `src/jobs/label.ts:22-27`
> **Why it matters:** `default: { const exhaustive: never = state; throw new Error(...) }` does provide compile-time exhaustiveness: adding an unhandled `JobState` makes the `never` assignment fail. However, this function receives only already-validated `JobState` values, so the throw is conceptually unreachable and adds a meaningless branch-coverage obligation without defining a real recovery path.
> **Evidence:** All current union members return from explicit cases, and the boundary parser rejects unknown states before calling this function. Unlike a bare `default: throw new Error("unreachable")`, the `never` assignment—not the throw—is what proves exhaustiveness to TypeScript.
> **Recommended direction:** Retain a compile-time `never`/`satisfies never` check using the repository's minimal post-switch pattern rather than an artificial runtime branch. If untyped values can actually reach this code, validate them at that boundary instead.

**Why the strong version is better:** It credits the `never` assignment as a real compile-time check, distinguishes it from an untyped throw-only default, and limits the criticism to unnecessary runtime behavior and coverage noise.

## 9. Coverage exclusion hides production code

**Severity:** MAJOR

**Bad finding wording**

> Coverage should be 100%; remove exclusions.

**Strong finding wording**

> **[MAJOR] The new coverage exclusion removes the conflict resolver from regression reporting**
> **Location:** `vitest.config.ts:34`
> **Why it matters:** `src/sync/conflict-resolver.ts` contains the data-preserving merge path changed by this PR, but the new glob excludes the entire `sync` directory. Coverage can stay green even if no test executes this risk-sensitive production code.
> **Evidence:** The exclusion drops the file from all four reported metrics; the PR adds no alternative suite or technical reason for excluding it.
> **Recommended direction:** Include production sync sources and add behavior tests for the changed conflict branches; exclude only genuinely generated or unreachable artifacts with rationale.

**Why the strong version is better:** It focuses on a risk-sensitive blind spot, not an arbitrary perfect-coverage target.

## 10. Moved tests are no longer discovered

**Severity:** BLOCKER

**Bad finding wording**

> Vitest config might need updating.

**Strong finding wording**

> **[BLOCKER] The moved authorization tests are not executed by any Vitest project**
> **Location:** `vitest.workspace.ts:7-18`
> **Why it matters:** This PR moves the only denial-path tests to `tests/security/`, while every configured project includes only `src/**/*.test.ts`. CI remains green with zero authorization tests, so the access-control change is unvalidated.
> **Evidence:** The test-list output contains no files under `tests/security`, and none of the workspace include globs match that directory.
> **Recommended direction:** Add the new directory to the appropriate Vitest project (and fail the job on an unexpectedly empty suite), then verify both allow and deny paths execute in CI.

**Why the strong version is better:** It proves that tests are absent from execution and relates that gap to the change's security risk.

## 11. Green CI hides a GET-then-unconditional-PUT race

**Severity:** BLOCKER

**Bad finding wording**

> There could be a race condition here.

**Strong finding wording**

> **[BLOCKER] Concurrent saves can silently overwrite a newer document version**
> **Location:** `src/documents/save.ts:55-79`
> **Why it matters:** Two clients can read version 7, produce different edits, and both issue unconditional PUTs; the later write succeeds and silently discards the first client's data. Green unit tests serialize the operations and do not exercise this interleaving.
> **Evidence:** `getDocument()` returns an ETag, but `putDocument()` is called without the adapter's `ifMatch` option. The API reports success for both writes.
> **Recommended direction:** Carry the observed version/ETag into a conditional write, return a typed conflict result, and test the two-writer interleaving.

**Why the strong version is better:** It supplies the exact interleaving, user-visible impact, existing concurrency mechanism, and missing test.

## 12. Documentation exists but omits the contract

**Severity:** MINOR

**Bad finding wording**

> Improve this TSDoc.

**Strong finding wording**

> **[MINOR] The exported timeout option does not document its unit or disable semantics**
> **Location:** `src/client/options.ts:14-17`
> **Why it matters:** The comment says only "Request timeout," while runtime treats the number as milliseconds and `0` as "no timeout." Callers can reasonably pass seconds or assume zero means immediate failure, producing materially different behavior.
> **Evidence:** `createDeadline()` compares the value directly with `Date.now()` deltas and skips deadline creation for zero.
> **Recommended direction:** Document the unit, zero behavior, and whether the timeout covers retries as part of the exported contract.

**Why the strong version is better:** It evaluates documentation usefulness and identifies the missing invariants instead of rewarding mere comment presence.

## 13. Session guard resets while non-cancellable work survives teardown

**Severity:** MAJOR

**Bad finding wording**

> The reload logic has a race condition.

**Strong finding wording**

> **[MAJOR] Re-enable resets the only exclusion guard while the old inspection is still pending**
> **Location:** `src/inspections/controller.ts:41-78`
> **Why it matters:** `busy` belongs to the enable-session object, but the host inspection call is non-cancellable and can outlive unload. Re-enabling creates a fresh session with `busy = false`, so a second inspection starts concurrently even though the operation contract allows only one at a time.
> **Evidence:** Unload discards the session without settling the pending host promise; re-enable constructs a new session before the first promise resolves. The nearby test resolves that promise before invoking the new command, so it exercises a safe order rather than the overlap.
> **Recommended direction:** Keep operation exclusion owned by the longer-lived operation owner, while keeping presentation/session identity separate so stale completion cannot update the new session. Leave the first promise pending during unload and re-enable, assert that the second operation does not start, then settle the first operation and verify normal operation resumes.

**Why the strong version is better:** It identifies the mismatched lifetimes, the harmful interleaving, the test's ordering gap, and a bounded direction that separates operation ownership from stale presentation.

## 14. Conditional request duplicates core ETag format knowledge

**Severity:** MAJOR

**Bad finding wording**

> The conditional-request parser has too many `if` statements. Use a `switch`.

**Strong finding wording**

> **[MAJOR] Transport re-implements core ETag extraction and hides a protocol decision matrix**
> **Location:** `apps/worker/src/http/conditional-request.ts:38-75` and `apps/core/src/etag.ts:12-31`
> **Why it matters:** The transport validates an `m3-<revision>` ETag and then manually slices it, while core already owns that format and its parser. The copies can drift so transport may extract a revision differently from the canonical validator. The early returns themselves are readable; the material defect is the second protocol-format owner, compounded by a decision matrix that makes future variants harder to audit.
> **Evidence:** The outer conflict/missing checks, mode-dependent `If-None-Match` branch, ETag validation, and revision extraction form separate protocol decisions. `apps/core/src/etag.ts:12-31` already owns `parseApplicationEtag` and the `m3-<revision>` format, while this transport function manually slices the validated string. The tests cover important examples but do not by themselves make the whole decision table apparent.
> **Recommended direction:** Preserve behavior and consider `classify -> exhaustive dispatch -> focused variant parsers` (or an equivalent explicit decision table). Reuse `apps/core/src/etag.ts:12-31` and replace `createApplicationEtag(ifMatch)` followed by `ifMatch.slice(4, -1)` with `parseApplicationEtag`, making core the single source of truth for validation and extraction. Do not introduce a generic header abstraction or centralize unrelated conditional-request policies. Add table-driven transport/core contract tests for every valid/invalid combination, including mode and mixed-condition interactions, and assert that the canonical parser rejects malformed revisions consistently.

**Why the strong version is better:** It identifies the exact transport/core duplication and drift risk, preserves the readable guards, and avoids treating `switch` or refactoring as an automatic requirement.

## 15. CORS policy duplicates the public route surface

**Severity:** MAJOR

**Bad finding wording**

> The CORS middleware has too many regexes. Replace the `if` with a `switch`.

**Strong finding wording**

> **[MAJOR] CORS independently defines a second public route and method policy**
> **Location:** `apps/worker/src/http/v2-cors.middleware.ts:14-42`; related route owners: `apps/worker/src/http/routes/v2.ts:8-67` and `apps/worker/src/openapi/v2.ts:20-91`
> **Why it matters:** The middleware's local route regexes and allowed-method lists describe which v2 endpoints are public, while Hono registration and OpenAPI already describe the deployed route surface. These independently editable copies can drift: a new route can be reachable but fail preflight, or an old method can remain allowed after runtime registration changes.
> **Evidence:** `v2-cors.middleware.ts` matches `/v2/...` paths and enumerates methods separately from the Hono route definitions and OpenAPI operations. The same API capability is therefore represented by three owners, not merely repeated text.
> **Recommended direction:** Make a typed route capability policy authoritative for route shape and allowed methods, then derive Hono registration metadata, CORS behavior, and OpenAPI operations from it (or use the repository's existing route-policy primitive). Do not replace `if` with `switch`, add a generic router abstraction, or centralize routes whose authorization or exposure policies intentionally differ. Add route-matrix tests covering every registered method/path and preflight, runtime, and generated-OpenAPI agreement, including an assertion that adding a route cannot silently omit its CORS policy.

**Why the strong version is better:** It identifies the semantic public-API rule and all owners, explains the drift failure, recommends one typed policy rather than a cosmetic control-flow change, and preserves intentionally different route policies.
