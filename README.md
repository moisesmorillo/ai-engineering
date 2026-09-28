# ai-engineering

Reusable assets for building and operating AI-assisted engineering workflows.

## Current structure

- [`skills/`](skills/) — portable, task-focused instructions for AI agents.
  - [`code-review`](skills/code-review/SKILL.md) — semantic review of code changes, tests, and accompanying documentation, with an initial TypeScript profile.
  - [`design-feasibility-review`](skills/design-feasibility-review/SKILL.md) — feasibility review of implementation-facing PRDs, ADRs, specifications, and architecture documents.

## Review routing

| Change | Review skill |
| --- | --- |
| Code, tests, or executable configuration, including their accompanying documentation | `code-review` |
| Implementation-facing PRD, ADR, specification, or architecture documentation only | `design-feasibility-review` |
| Code and implementation-facing design documentation together | Both skills |
| Purely editorial documentation | Neither by default |

Each skill reviews the full resulting change in its scope. A design correction requires another feasibility pass over the whole design, including resource budgets affected by the correction.

Skills provide reusable defaults, not universal project policy. When using one, follow the target repository's `AGENTS.md`, contributing guide, architecture documentation, ADRs, CI configuration, and established conventions first.

Future additions may include `agents/`, `plugins/`, `prompts/`, `evals/`, and `templates/`. These directories will be added when they contain useful assets rather than as empty placeholders.

## Contributing

Keep assets self-contained, broadly reusable, and explicit about when project-local rules take precedence. Avoid unnecessary runtime dependencies and project-specific assumptions.

## License

[MIT](LICENSE)
