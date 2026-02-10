# codex-boilerplate

Reusable scaffolding for Codex-style agent workflows.

## Contents

- `AGENTS.md`: repository-wide agent rules and development principles.
- `codex/skills/`: reusable skills (`loop`, `debug`, `inject-context`).
- `.codex/skills/`: optional mirror of `codex/skills/` for tools that expect it.
- `.mcp.json`: optional MCP configuration (includes a Figma MCP example).
- `docs/` and `prd/`: lightweight documentation layout.

## Docs Conventions

- `docs/tech/`: long-lived technical contracts (architecture, interfaces, rules, guarantees).
- `prd/`: task-level PRDs tied to real deliverables (scope, acceptance criteria, test/verification plan).

Rule of thumb:

- If it is a stable, cross-cutting agreement, put it in `docs/tech/`.
- If it is a one-off deliverable, put it in `prd/` and reference `docs/tech/` where relevant.

## Usage

1. Copy these files into the root of your repository.
2. Keep skills canonical in `codex/skills/`.
3. If your tooling expects `.codex/skills/` or per-subproject skills, mirror them and keep the copies synchronized.

## MCP (Optional)

`.mcp.json` contains an example Figma MCP server configuration. It expects an environment variable like `FIGMA_READ_TOKEN` to be set locally.

