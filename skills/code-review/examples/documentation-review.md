# Documentation Review Examples

These illustrative scenarios exercise the [TypeScript declaration policy](../profiles/typescript.md#documentation) and [semantic documentation audit](../checklists/semantic-review.md#documentation-completeness-and-quality). Paths, symbols, and convention evidence are hypothetical, not findings about this skill repository. Code excerpts omit unrelated implementation.

## A — Valid PR finding for internal compatibility helpers

A PR materially changes `apps/plugin/src/state/device-state-v2.codec.ts`. Repository instructions require useful TSDoc for named declarations, and representative source follows that convention. The exported decoder explains the historical format, but its private/free converters do not document their distinct compatibility obligations. Existing tests establish exact reconstruction, not repair or migration.

> **[MINOR] Internal compatibility contracts are undocumented**
> **Location:** `apps/plugin/src/state/device-state-v2.codec.ts` — `convertDeviceState`, `convertUnresolvedMutation`, `convertStagedHandoff`.
> **Why it matters:** Maintainers must reconstruct which conversions preserve historical predicates, revisions, retries, and handoff authority. The exported decoder's comment does not explain each helper's obligations, increasing the risk of later “normalization” across the frozen compatibility boundary.
> **Evidence:** The changed helpers have no TSDoc; the persistence ADR and historical round-trip fixtures require exact reconstruction. The repository's named-declaration convention applies to internal code too. Current behavior appears sound.
> **Recommended direction:** Add useful contracts at these helpers, describing the preserved state and forbidden reinterpretation. Reuse authoritative contract references where supported rather than copying the decoder's prose verbatim. Keep the fix within this changed compatibility path.

This is one bounded MINOR finding, not one per helper and not an automatic MAJOR because persistence is involved. Inspect related lifecycle, path-state, acknowledgement, desired-state, transferable-acknowledgement, and identifier/precondition converters when the changed path depends on them. Private placement alone does not require extraction.

## B — Invalid noisy review and empty-comment “fix”

**Do not produce:**

```text
[MAJOR] convertLifecycle lacks a docstring.
[MAJOR] convertPathState lacks a docstring.
[MAJOR] convertAcknowledgement lacks a docstring.
... 200 more missing-comment findings ...
```

This omits convention evidence, missing meaning, and impact; it exaggerates severity and fragments one pattern. Group the evidence as in A or C. Do not fix the noise by ignoring all private declarations.

These comments are present but semantically empty for the stated contracts:

```ts
/** Converts state. */
function convertDeviceState(dto: DeviceStateDto): DeviceState { /* ... */ }

/** User ID. */
type UserId = string;

/** Gets the user. */
function getUser(id: UserId): User { /* ... */ }
```

If evidence shows `UserId` is tenant-scoped rather than global, or `getUser` returns a cached observation rather than authoritative security evidence, document that meaning. For the converter, describe frozen rehydration and prohibited repair/reinterpretation as in the profile. Do not invent identity or lookup guarantees merely to improve prose.

## C — Repository-wide aggregated finding

The user requests a baseline review. No PR/diff exists. The reviewer records the baseline revision, inventories production roots `apps/plugin/src`, `apps/worker/src`, and `packages/core/src`, and excludes generated clients/vendor paths with configuration evidence. Declaration searches across all roots plus reading codec, coordinator, schema, and domain modules reveal the same gap; unrelated documented modules are inspected as counterexamples. Deep semantic reading is sampled, not claimed exhaustive.

> **[MINOR] Internal semantic contracts are systematically undocumented**
> **Locations:** `apps/plugin/src/state/device-state-v2.codec.ts` (`convertUnresolvedMutation`); `apps/plugin/src/sync/coordinator.ts` (`applyAcknowledgement`); `apps/worker/src/persistence/handoff.schema.ts` (`handoffSchema`).
> **Why it matters:** Exported APIs have useful contracts, but internal converters, state helpers, and persisted schemas leave compatibility and state-machine invariants implicit. Future reviews must repeatedly reconstruct intent from code and tests.
> **Evidence:** Repository-wide declaration searches and representative reads across these roots show missing helper/schema TSDoc and comments that only paraphrase names. Public decoders and ports demonstrate TSDoc use; there is no evidenced internal-code exemption. The persistence/migration tests establish the semantics these comments omit.
> **Recommended direction:** Document named internal semantic declarations at their owners and capture the expectation in repository conventions. Address this pattern by owner, distinguishing missing comments from empty ones; do not require unrelated refactors or generic utilities.

Report a few such owner/pattern findings, not every symbol. Include roots searched, representative files read, exclusions, and uninspected areas in the validation summary. A separate materially misleading security contract may deserve its own higher-risk finding rather than being hidden in this aggregate.

## D — Runtime schema and DTO boundary

A named Zod schema decodes frozen historical state. Apply the profile's [schema example](../profiles/typescript.md#zod-and-other-runtime-schemas): the schema-level TSDoc identifies its persisted version, migration-only role, and prohibition on widening acceptance. The inferred `DeviceStateDto` alias documents the validated DTO **before** domain rehydration, not a live domain object.

**Expected:** Missing schema TSDoc and an undocumented architecturally meaningful DTO alias belong in the compatibility finding when they share its owner/remediation. Verify the prose against parsing, unknown-key handling, transformations, and compatibility tests. A `.strict()` call alone does not establish correct versioning. Do not comment on every `.object()`, `.string()`, `.enum()`, or field whose meaning is obvious. An incidental local `z.infer` alias without a separate architectural role does not require its own comment.

## E — Private state method versus mechanical delegation

A class-level description says “Coordinates synchronization.” That does not describe this method's fencing, mutation, and effect obligations. Assuming code and tests support them, useful TSDoc might be:

```ts
/**
 * Commits acknowledgement evidence only for the active lifecycle generation.
 *
 * Stale completions must not mutate durable state or release a newer generation's
 * reservation. An uncertain remote effect remains unresolved for reconciliation.
 */
private applyAcknowledgement(result: MutationResult, generation: number): void {
  /* ... */
}

/** Delegates byte accounting to the transport's canonical UTF-8 size contract. */
private byteLength(value: string): number {
  return this.transport.byteLength(value);
}
```

**Expected:** The state method needs its distinct semantic contract regardless of `private`, `protected`, or `#private` syntax. The genuinely mechanical delegate can use one useful line; it does not need a copied lifecycle essay. Under the repository's all-named-declarations convention, both enter the inventory, but the trivial helper's isolated omission is at most NOTE/NIT, usually not a standalone finding. Anonymous callbacks, local values, destructuring, generated/vendor code, and obvious one-off literals do not acquire doc obligations merely by existing.

## F — Corrective closure tests meaning, not delimiters

For A's prior MINOR, non-blocking finding:

| Corrective result | Disposition | Evidence required |
|---|---|---|
| Adds only `/** Converts state. */` to all helpers | `still open — partially remediated` | Blocks exist but still omit the compatibility obligations. |
| Copies “never mutates” onto a helper that mutates its input | `still open` | Final code contradicts the new guarantee; reassess impact rather than reward more prose. |
| Documents only the exported decoder or one helper | `still open — partially remediated` | Remaining named examples and the agreed pattern scope still lack useful contracts. |
| Adds accurate per-role contracts and corrects stale text after a code change | `fixed` | Final code, callers, governing compatibility rules, and tests agree; the original pattern was rechecked across its agreed scope. |

Use the existing finding ledger and verdict rules. Non-blocking residual debt permits `APPROVE WITH NOTES` when acknowledged; an unresolved prior blocker still requires `REQUEST CHANGES`. Documentation alone does not close a separate structural/stateful finding.

## G — Combined acceptance scenario

Apply the modified skill to a TypeScript file under the named-declaration TSDoc convention:

| Declaration | Expected audit result |
|---|---|
| Well-documented exported decoder | Useful, provided its claims match code/tests; no documentation finding. |
| Undocumented internal conversion helpers | Missing contracts; include compatibility/predicate/revision/handoff obligations supported by evidence. |
| Undocumented private state-machine methods | Missing distinct transition, precondition, mutation, lifecycle, or effect contracts. |
| Undocumented named persisted-state Zod schemas | Missing schema-level compatibility/boundary documentation. |
| Undocumented architecturally meaningful inferred DTO aliases | Documentation expected to distinguish DTO from domain state. |
| Trivial `byteLength()` | Inventory it; concise unit/encoding/delegation semantics suffice, not a high-severity standalone finding. |
| `/** Converts state. */` | Present but semantically empty; does not satisfy the contract audit. |

**Expected synthesis:** Group the shared missing/empty-contract pattern into a bounded MINOR finding with concrete examples, unless independent evidence justifies a different severity. A PR focuses on introduced/materially changed declarations and materially relevant nearby gaps; a baseline review searches across production roots before asserting systemic debt. A strong, evidenced alternative documentation convention changes the required format/coverage, not the obligation to evaluate semantic clarity.
