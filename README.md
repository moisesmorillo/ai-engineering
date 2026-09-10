# ai-engineering

Reusable assets for building and operating AI-assisted engineering workflows.

## Current structure

- [`skills/`](skills/) — portable, task-focused instructions for AI agents.
  - [`code-review`](skills/code-review/SKILL.md) — semantic code and pull-request review, with an initial TypeScript profile.

Skills provide reusable defaults, not universal project policy. When using one, follow the target repository's `AGENTS.md`, contributing guide, architecture documentation, ADRs, CI configuration, and established conventions first.

Future additions may include `agents/`, `plugins/`, `prompts/`, `evals/`, and `templates/`. These directories will be added when they contain useful assets rather than as empty placeholders.

## Contributing

Keep assets self-contained, broadly reusable, and explicit about when project-local rules take precedence. Avoid unnecessary runtime dependencies and project-specific assumptions.

## License

[MIT](LICENSE)
