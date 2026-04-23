# Skills

This repository has local Codex skills under `codex/skills/`.

## Available skills

- `loop`: Close-the-loop implementation workflow. Use when you must finish code changes with CLI-based verification on real infra (test/lint/build/e2e) before stopping. (file: `codex/skills/loop/SKILL.md`)
- `debug`: Root-cause-first debugging workflow. Use for bug reports or unexpected behavior where diagnosis evidence should come before any fix. (file: `codex/skills/debug/SKILL.md`)
- `inject-context`: Context-injection workflow that auto-selects project/domain docs and code architecture before executing the actual task. Use when requests are broad or cross-domain. (file: `codex/skills/inject-context/SKILL.md`)

## How to use skills

- Trigger a skill by name (`loop`, `debug`, `inject-context`) or when the task clearly matches its description.
- Use the minimal set of skills needed for the turn.
- Read `SKILL.md` first, then load only referenced files required for the current task.
- If a referenced file is missing, continue with best-effort fallback and state what is missing.
- Treat `codex/skills/` as the canonical source. If your tooling expects it, mirror skills into `.codex/skills/` and/or `<subproject>/codex/skills/` and keep them synchronized.

## Docs Layout (Required)

- `docs/`: long-lived documentation. Put cross-cutting technical contracts in `docs/tech/`.
- `prd/`: task-level PRD documents that map to real deliverables.

## Parallel Agent Execution

- Assume parallel agents may run in the same repository at the same time.
- Parallelize independent read/search/verification tasks aggressively, but do not edit the same file from multiple agents concurrently.
- Before editing, define the exact target file list and avoid scope creep.
- After each edit batch, re-check `git status --short` and `git diff --name-only` for unexpected overlap.
- If unexpected edits from another agent appear in files being touched, stop and coordinate. Do not revert other agent changes.
- For broad requests, split work by domain (e.g. `server`, `web`, `mobile`, `packages`, `apps/*`) and merge only after each slice is verified.

## Commit Discipline

- Always work and commit in atomic units. Break every task into logical atomic steps and leave an atomic commit immediately after each step is completed.
- Do not finish a batch of work and split commits afterward. Commit incrementally as each atomic unit reaches a complete, verified state.
- Keep commits atomic: one logical change per commit, and include only paths touched for that change.
- Always run `git status --short` before staging/committing.
- For tracked files, commit with explicit paths: `git commit -m "<scoped message>" -- path/to/file1 path/to/file2`.
- For brand-new files, use: `git restore --staged :/ && git add "path/to/file1" "path/to/file2" && git commit -m "<scoped message>" -- path/to/file1 path/to/file2`.
- Verify staged scope before commit: `git diff --cached --name-only`.
- Never bundle unrelated fixes in the same commit, even if they are in nearby code.
