# TypeScript Review Profile

Apply this profile in addition to the language-agnostic core. First inspect the repository's `tsconfig`, package boundaries, framework setup, scripts, lint/format configuration, test configuration, and documented conventions. Prefer precise, readable types over maximal type cleverness.

## Typing

- Trace every `any`: is it required at an external boundary, narrowly contained, and validated, or does it disable checking through the call graph?
- Ensure broad `unknown` values are narrowed before property access, persistence, branching, or boundary crossing. `unknown` is not safety if it is merely cast onward.
- Challenge unchecked `as` casts and double casts. Look for runtime evidence that justifies the asserted type.
- Treat non-null assertions as claims that need a proven invariant; do not let `!` hide an actually reachable missing state.
- Check generic defaults and omitted framework generic parameters for silent degradation to broad types.
- Do not use `Record<string, unknown>` or open index signatures when the object has a known closed shape.
- Give duplicated union literals and shared enum-like values one authoritative exported type/value definition when they represent one policy.
- Type protocol statuses, headers, media types, and error codes strongly enough to prevent drift without making simple code opaque. Inspect method/status/action literals collectively when they encode one policy matrix even if each occurs only once; do not introduce types or constants merely to hide ordinary local literals.
- Model closed states with discriminated unions where that prevents invalid boolean combinations or enables exhaustive handling.
- For a closed union, classify or narrow its discriminant once at the semantic owner when practical. Repeated checks of the same tag across distant validator branches or callers can hide an omitted variant or duplicate policy; prefer exhaustive dispatch when it materially exposes completeness, but do not replace every readable local `if` with a `switch`.
- Verify type guards actually prove their predicate and do not rely on unchecked shape assumptions.

Do not demand advanced conditional types, branded types, or generic abstractions unless their safety benefit outweighs their cognitive cost.

## Imports and modules

- Use `import type` where required by the repository's compiler/linter/module settings and where it clarifies type-only dependency edges.
- Follow existing path-alias conventions; do not force aliases on a repository that intentionally uses relative imports.
- For cross-package use, prefer the target package's public exports rather than private internal paths.
- Flag relative imports that escape into another project/package and bypass its API.
- Check for new circular dependencies, especially cycles hidden by type/value import confusion or barrel exports.
- Confirm ESM/CommonJS and file-extension choices match the target build and runtime.
- For exported interfaces, types, ports, and protocol constants, audit semantic ownership and import ergonomics: do not move a contract solely because it shares a file with its implementation, but flag a separately reused or architecturally distinct contract when consumers must import concrete infrastructure to consume it, implementation details leak into the public surface, or placement creates a circular/awkward dependency. Preserve colocated private/local types when clearer.

## Hono and Cloudflare Workers

Apply this section only to Hono or Worker code.

- Inspect the installed Hono version's exact API defaults before claiming type degradation. Do not assume every omitted generic is broad; flag only defaults, annotations, or composition points that actually weaken environment, bindings, variables, route schema, or context typing.
- Ensure Worker secrets and bindings have precise environment types rather than broad string maps or `any`.
- Keep Hono request/context/response types in the transport layer when the project maintains application/core boundaries.
- Flag handlers that call database, object-store, queue, or other infrastructure adapters directly when an application service/port is the intended boundary.
- Verify response status types and headers derive from or conform to authoritative protocol definitions.
- Compare route registration, OpenAPI declarations, runtime validation, handler behavior, and deployed Worker bindings for drift.
- Check execution-lifecycle assumptions such as deferred work, cancellation, streaming, and platform limits where relevant.

These are framework-specific checks, not universal TypeScript policy.

## OpenAPI and runtime schemas

- Compare runtime validation with generated/published API documentation.
- Replace broad string schemas when a meaningful domain constraint is known and enforced by runtime behavior.
- Avoid separate protocol definitions that can drift across schemas, handlers, clients, and tests.
- Verify generated-client assumptions—required fields, nullability, statuses, media types, and headers—match runtime behavior.
- Ensure transformations and refinements are represented consistently at input, domain, and output boundaries.

## Documentation

This section owns TypeScript declaration coverage and TSDoc/JSDoc quality rules. Use the [semantic documentation audit](../checklists/semantic-review.md#documentation-completeness-and-quality) for audit scope, repository-wide aggregation, severity, and corrective closure.

**In TypeScript repositories using TSDoc/JSDoc-style documentation, every named semantic declaration should have useful TSDoc unless a strong, evidenced repository convention says otherwise.** Discover that convention from applicable instructions, tooling, and representative source across relevant packages. Cite an intentional alternative (including established inherited-contract documentation); widespread omissions alone do not prove an exemption. Private visibility, small size, or an apparently obvious name does not exempt a named declaration from review. If the repository intentionally uses another established documentation convention, assess completeness and semantic quality in that convention rather than imposing TSDoc.

### Declaration inventory

Inventory named declarations in changed TypeScript files, not only exported APIs or structural candidates. Prioritize introduced or materially changed declarations; inspect nearby existing declarations when the changed contract depends on them or they reveal a systemic violation materially affecting the change. Do not turn a focused PR into an unrelated full-repository cleanup. In explicit repository-wide mode, inventory across production source roots and apply the semantic checklist's completeness audit.

Evaluate documentation for:

- exported and internal/free functions, including named function-valued bindings such as `const decode = (...) => ...`;
- public, protected, and private methods (including `#private` methods), named object methods, and accessors;
- classes, and constructors when construction has non-obvious semantics;
- interfaces, type aliases, ports/capabilities/contracts, discriminated unions, and branded/domain IDs, even when implementation-local;
- enums, enum-like authoritative constant objects, and meaningful domain/runtime constants;
- named runtime schemas and architecturally meaningful inferred DTO aliases; and
- properties/fields whose meaning, units, authority, lifecycle, nullability, security significance, or valid combinations are not obvious.

Do not require separate TSDoc for anonymous callbacks/lambdas merely because they exist, ordinary local variables, trivial destructuring, individual schema-chain calls, obvious one-off literals, generated code, or third-party/vendor code. A local binding that defines a named function, semantic schema, or domain constant is not exempt merely because it uses `const` or lives inside another function. A local `z.infer` alias without a distinct architectural role does not need a mechanical separate comment.

### Semantic quality, not comment count

Classify each in-scope declaration as **missing**, **present but semantically empty**, or **useful contract documentation**; record evidenced exemptions separately. Present but stale/incorrect comments also fail review—identify the contradiction, not just the presence of a block. No tag count or comment length establishes usefulness.

Useful documentation captures what a future maintainer cannot safely infer from syntax alone, as applicable: responsibility and semantic role; ownership and authority; invariants; preconditions and postconditions; state transitions; units; side effects and mutation; concurrency/lifecycle, cancellation and settlement; security/privacy; failure/effect certainty; resource ownership/release; persistence compatibility; downgrade/migration boundaries; and what the declaration intentionally must **not** do. Use `@throws` and other tags when they clarify a real contract, not to repeat parameters or obvious return types. Verify claims against implementation, callers, tests, and governing contracts; do not invent guarantees to fill a comment.

`/** Gets the user. */`, `/** Converts state. */`, or `/** User ID. */` is semantically empty when lookup authority, compatibility, or identity scope is the actual contract. Neither paraphrasing a name nor narrating trivial implementation mechanics resolves a finding. Concise comments can be sufficient: document the relevant role or constraint, not every item in the list above.

### Internal helpers and private methods

Explicitly audit private/protected methods implementing state transitions, persistence, retry policy, security decisions, validation, reconciliation, lifecycle fencing, effect classification, migration, or protocol mapping. A class-level comment does not cover a method's distinct preconditions/effects. Free/internal conversion helpers also need their own semantic contracts: an exported decoder's good TSDoc does not automatically document its helper tail.

For historical or persisted codecs, inspect converters for legacy versioned state, persisted identifiers, predicates, revisions, effects, and other compatibility-sensitive fields. Document whether each converter performs exact compatibility-preserving reconstruction, a migration-only transformation, or version/downgrade fencing. Where the contract requires exact rehydration, make clear that repair, normalization, and reinterpretation are forbidden. Do not copy the same broad promise onto every helper without checking its actual role.

For example, replace `/** Converts legacy state. */` with evidenced contracts such as:

```ts
/**
 * Rehydrates a validated legacy-version DTO into its exact domain representation.
 *
 * Performs persisted identifier reconstruction only. It must not migrate, repair,
 * normalize, or reinterpret the historical state.
 */
function convertLegacyState(dto: LegacyStateDto): HistoricalState { /* ... */ }

/**
 * Reconstructs one validated persisted operation without changing its action,
 * original predicate, revision, retry counters, or effect-recovery semantics.
 *
 * This compatibility-preserving converter must never derive replacement evidence
 * from newer state.
 */
function convertPersistedOperation(dto: PersistedOperationDto): PersistedOperation { /* ... */ }
```

For truly mechanical private delegation, a short one-line TSDoc identifying the delegated contract/owner is acceptable. Where the repository supports it, use a resolvable inherited-contract reference instead of copying interface prose; verify identical semantics and document implementation-specific effects separately. Without such an evidenced convention, do not silently exempt implementations. A trivial named helper such as `byteLength()` still enters the inventory; a short unit/encoding contract is enough, and its isolated omission is not a high-severity standalone finding.

### Types, interfaces, and constants

For named types/interfaces, explain whether the representation is domain, DTO, persistence, protocol, or adapter-local, and its owner. Cover units, valid state combinations, lifecycle, security meaning, compatibility/versioning obligations, and whether fields carry authoritative evidence or merely observations where applicable. Clarify whether `null` means absent, unknown, not-applicable, or pending. Domain/branded IDs need identity scope and relevant validity/authority semantics, not just “ID.” Private/local interfaces are not exempt.

Document meaningful constants and authoritative enum-like objects with their semantic role, policy owner, units, or compatibility constraints—not merely their numeric/string values. Review field-specific semantics at the field when they would otherwise be lost; do not demand comments on every obvious property or enum member.

### Zod and other runtime schemas

A named schema representing persisted state, protocol requests/responses, configuration, handoff payloads, security-sensitive validation, migration compatibility, or domain state is a semantic contract regardless of export visibility. Document its purpose, boundary/owner, accepted version, and constraints that must survive edits. Document the schema as a whole, not every `.object()`, `.string()`, or `.enum()` call; add field-level documentation only for non-obvious semantics.

Illustrative excerpts (the comment must match the actual schema and compatibility tests):

```ts
/**
 * Frozen historical persisted-state schema for one legacy format version.
 *
 * Used only by a migration decoder. It must not be widened to accept fields from
 * other persisted versions or future formats.
 */
const legacyPersistedStateSchema = z.object({ /* frozen legacy fields */ }).strict();

/** Validated legacy persisted-state DTO before domain rehydration. */
type LegacyPersistedStateDto = z.infer<typeof legacyPersistedStateSchema>;
```

Require a separate inferred-type comment when the alias has a meaningful architectural role, such as distinguishing a validated persisted DTO from rehydrated domain state. Do not mechanically document every local `z.infer`. Schema TSDoc must agree with actual parsing, unknown-key, transform/refinement, and version behavior; a claim of frozen acceptance does not make a permissive parser safe.

See [documentation review examples](../examples/documentation-review.md) for PR, baseline, private-method, schema, and corrective calibration.

## File conventions

Enforce these only when the target repository defines them:

- `*.types.ts` contains type-level declarations only;
- `.constants.ts` contains runtime constants;
- `.d.ts` is reserved for ambient declarations or module augmentation.

Also inspect local conventions for barrels, colocated tests, generated files, casing, and public entrypoints. A preference not established by the repository is not a finding.

## Tooling and diagnostics

Inspect actual versions and configuration before enforcing tool-specific rules.

- **TypeScript strict mode:** verify the relevant project/reference is included and strict options are not weakened or bypassed locally.
- **Biome:** determine whether it owns formatting, linting, import organization, and which paths/rules are excluded.
- **Oxlint / type-aware tsgolint:** distinguish syntax-only checks from type-aware diagnostics and confirm the intended command/config runs in CI.
- **Vitest:** inspect workspace/project configuration, include/exclude globs, environments, setup files, coverage provider, source inclusion, and thresholds; verify moved or newly named tests are discovered.
- **mise:** prefer the repository's pinned tasks/tool versions when mise is the canonical entrypoint; flag direct invocations only when they bypass required setup or checks.

Also inspect deprecation warnings, framework diagnostics, editor diagnostics, and generated schema/client warnings that the canonical CI path may not surface.
