# Repository Guidelines

## Project Structure & Module Organization

Fulmen is a multi-agent harness for Codex and Claude. The main implementation
files live under `implementations/`: `implementations/codex/agents/*.toml`
contains Codex role definitions, `implementations/codex/skills/fulmen/`
contains the Codex skill package, and `implementations/claude/agents/*.md`
plus `implementations/claude/skills/fulmen/` mirror the Claude setup. Project
documentation lives in `docs/`, and repository utilities live in `scripts/`.
There are no compiled assets or application source directories beyond these
agent, skill, documentation, and script files.

## Build, Test, and Development Commands

- `./scripts/validate`: runs repository consistency checks for required role
  files, skill references, TOML fields, YAML frontmatter, and verifier safety
  rules.
- `./scripts/fulmen-live status`: shows whether live `~/.codex` and `~/.claude`
  targets are linked to this repo.
- `./scripts/fulmen-live install`: links this repo into the live Codex and
  Claude config locations, backing up unmanaged existing files first.
- `./scripts/fulmen-live doctor`: runs validation and verifies all live links.
- `./scripts/fulmen-live uninstall`: removes only repo-owned live symlinks.

Run `./scripts/validate` before committing changes to roles, skills, docs that
describe required behavior, or installer scripts.

## Coding Style & Naming Conventions

Use Bash with `set -euo pipefail` for scripts. Keep shell functions small,
quote variable expansions, and prefer explicit status messages. Codex agent
files use TOML with `name`, `description`, `sandbox_mode`, and triple-quoted
`developer_instructions`. Claude agent files use YAML frontmatter followed by
Markdown. Role names are fixed: `planner`, `critic`, `explorer`, `worker`,
`verifier`, and `assembler`.

## Testing Guidelines

This repository uses script-based validation rather than a unit test framework.
Add validation rules to `scripts/validate` when introducing a new required file,
role invariant, or safety contract. Keep checks deterministic and local; they
should not depend on live symlinks unless they belong in `fulmen-live doctor`.

## Commit & Pull Request Guidelines

Recent commits use concise, imperative subject lines such as `Document Fulmen
pathworking model` and `Preserve user intent across Fulmen delegation`. Follow
that style: one clear sentence, no trailing punctuation. Pull requests should
describe the changed roles or scripts, explain any behavior or safety-contract
changes, and include the output of `./scripts/validate`; include
`./scripts/fulmen-live doctor` output when live-link behavior changes.
