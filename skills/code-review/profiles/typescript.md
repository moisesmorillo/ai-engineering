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

Where the project uses TSDoc:

- document a public/exported declaration at its authoritative abstraction when callers need non-obvious invariants, units, side effects, security semantics, ownership/lifecycle, concurrency behavior, failure/effect semantics, resource-release requirements, or obligations;
- nontrivial internal contracts should explain invariants or lifecycle expectations when not evident from types;
- type-only declarations should explain units, meaning, valid combinations, or security implications when non-obvious;
- `@throws`, side effects, mutation, nullability, and async lifecycle should be documented when they are part of the caller contract; and
- do not duplicate boilerplate on an implementation method that satisfies an already well-documented interface unless implementation-specific semantics differ.

Do not require TSDoc universally when the repository uses another documented convention, and do not request comments that only restate names or types.

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
