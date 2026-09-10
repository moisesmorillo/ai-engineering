# Examples of Review Findings

These examples demonstrate evidence and impact, not universal project rules. Locations and names are illustrative.

## 1. Framework default generic weakens typing

**Severity:** MAJOR

**Bad finding wording**

> Add more Hono types here.

**Strong finding wording**

> **[MAJOR] The default Hono environment type hides missing Worker bindings**
> **Location:** `src/http/app.ts:12`
> **Why it matters:** `new Hono()` leaves `c.env` weakly typed, while this route reads `OBJECTS` and `SIGNING_KEY`. A misspelled or undeclared binding can pass typechecking and fail only in the deployed Worker.
> **Evidence:** `upload.ts` reads both bindings from the context, but the app instance does not supply the repository's `WorkerEnv` generic.
> **Recommended direction:** Instantiate the app with the established environment type and propagate it through route composition so binding access is checked.

**Why the strong version is better:** It identifies the lost guarantee, the runtime failure, and the bounded correction instead of demanding unspecified "more types."

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

**Why the strong version is better:** It identifies the missing state and externally observable consequence rather than suggesting a `default` that could hide future omissions.

## 8. Artificial runtime default for compile-time exhaustiveness

**Severity:** MINOR

**Bad finding wording**

> This default is ugly.

**Strong finding wording**

> **[MINOR] The synthetic default creates an unreachable runtime branch and weakens exhaustiveness**
> **Location:** `src/jobs/label.ts:22-24`
> **Why it matters:** Every `JobState` member is already handled, but `default: throw new Error("unreachable")` prevents the compiler pattern used elsewhere from identifying a newly added unhandled state and creates a meaningless uncovered branch.
> **Evidence:** The repository's closed-state helpers use a `never` exhaustiveness check after explicit cases; no untyped input reaches this function.
> **Recommended direction:** Use the established compile-time `never` check without adding an artificial runtime case. Validate at the external boundary if untyped states can actually enter.

**Why the strong version is better:** It explains both the type-safety and coverage effects and conditions the advice on boundary validation.

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
