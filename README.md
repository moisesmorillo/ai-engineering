# ai-engineering

Reusable assets for building and operating AI-assisted engineering workflows.

## Current structure

- [`skills/`](skills/) — portable, task-focused instructions for AI agents.
  - [`code-review`](skills/code-review/SKILL.md) — semantic code and pull-request review of implementation, tests, executable configuration, and accompanying documentation, with an initial TypeScript profile.
  - [`design-feasibility-review`](skills/design-feasibility-review/SKILL.md) — feasibility review of implementation-facing PRDs, ADRs, specifications, and architecture documents.
- [`odd/`](odd/) — task records for work delivered in this repository: objective, scope, acceptance criteria, and validation evidence. They document decisions, not live session state.

## Review routing

| Change | Review skill |
| --- | --- |
| Code, tests, or executable configuration, with accompanying documentation that is not an implementation-facing design document | `code-review` |
| Implementation-facing PRD, ADR, specification, or architecture documentation only | `design-feasibility-review` |
| Code together with an implementation-facing PRD, ADR, specification, or architecture document | Both skills |
| Purely editorial documentation | Neither by default |

An implementation-facing design document is never "accompanying documentation": it always takes the design route, even when it ships in the same change as code. Each skill reviews the full resulting change in its scope and defines its own corrective re-review rule. When both skills run, the more restrictive verdict governs the merge decision: `not ready` or `REQUEST CHANGES` from either skill blocks, regardless of the other skill's result.

Skills provide reusable defaults, not universal project policy. When using one, follow the target repository's `AGENTS.md`, contributing guide, architecture documentation, ADRs, CI configuration, and established conventions first.

Future additions may include `agents/`, `plugins/`, `prompts/`, `evals/`, and `templates/`. These directories will be added when they contain useful assets rather than as empty placeholders.

## Contributing

Keep assets self-contained, broadly reusable, and explicit about when project-local rules take precedence. Avoid unnecessary runtime dependencies and project-specific assumptions.

## License

[MIT](LICENSE)
