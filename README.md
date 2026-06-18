# Fulmen

Fulmen is a multi-agent harness for Codex and Claude. It keeps planning, critique,
exploration, implementation, verification, and final assembly as separate roles so
each task has a clear owner and review path.

This repository is the canonical source for the live Fulmen setup on this
machine. Files under `~/.codex` and `~/.claude` should be linked from this repo
instead of edited directly.

## Layout

```text
implementations/codex/   Codex skill and subagent role files
implementations/claude/  Claude skill and subagent role files
docs/                    Architecture, role, setup, and verifier notes
scripts/fulmen-live      Symlink installer and status tool
scripts/validate         Repository validation checks
```

## Install

```bash
./scripts/fulmen-live install
./scripts/fulmen-live doctor
```

The installer creates symlinks into:

- `~/.codex/skills/fulmen`
- `~/.codex/agents/*.toml`
- `~/.claude/skills/fulmen`
- `~/.claude/agents/*.md`

Existing unmanaged files are moved into timestamped backups under
`~/.local/share/fulmen/backups/` before links are created.

## Roles

- `planner`: creates the ordered approach.
- `critic`: finds problems in the plan.
- `explorer`: maps the relevant code or system surface.
- `worker`: makes the scoped implementation change.
- `verifier`: independently checks the completed change.
- `assembler`: combines outputs into a coherent final deliverable when needed.

## Validation

```bash
./scripts/validate
./scripts/fulmen-live status
```

Run validation before installing and after any role or skill changes.
