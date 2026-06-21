# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Fulmen is a multi-agent orchestration **harness**, not an application. The repo
contains prompt/config artifacts (Markdown, TOML, YAML) plus Bash tooling. There
is no build step, no compiler, and no language test suite — "correctness" here
means the prompt files are structurally valid and the two platform
implementations stay in parity. This repo is the canonical source for the live
Fulmen setup on this machine; `~/.codex` and `~/.claude` are populated by
symlinks back into it, so **never edit files under `~/.codex` or `~/.claude`
directly** — edit them here and re-link.

## Commands

```bash
./scripts/validate              # structural + parity checks; run before install and after ANY role/skill edit
./scripts/fulmen-live status    # show live symlink state (linked / repo / foreign / file / missing)
./scripts/fulmen-live install   # link this repo into ~/.codex and ~/.claude (backs up unmanaged files first)
./scripts/fulmen-live doctor    # runs validate, then confirms every live target points back here
./scripts/fulmen-live uninstall # remove only symlinks that point into this repo
```

`validate` is the test suite. It exits non-zero on any failure and is the gate
to satisfy after editing any agent or skill file. `doctor` is `validate` plus
live-link verification. To check a single concern, read the relevant
`require_contains` / `reject_*` assertion in `scripts/validate` — the checks are
explicit and greppable rather than parameterized.

`install` moves any pre-existing non-repo target into
`~/.local/share/fulmen/backups/<timestamp>/` before linking; repo-owned symlinks
are updated in place.

## Architecture

### Two parallel implementations, one contract

The same harness is implemented twice and **must be kept in sync**:

- `implementations/codex/` — TOML agents (`name`, `description`, `sandbox_mode`,
  `developer_instructions`) and a SKILL that drives the
  `multi_agent_v1.spawn_agent` / `wait_agent` / `close_agent` tools. Codex
  agents are **persistent** and must be explicitly closed after use.
- `implementations/claude/` — Markdown agents (YAML frontmatter: `name`,
  `description`, `model`, `maxTurns`, `disallowedTools`) and a SKILL that drives
  the `Agent` tool with `subagent_type`. Claude subagents are **one-shot** (no
  close needed); a still-running one can be continued with `SendMessage`.

`scripts/validate` encodes the parity contract: for every role it requires the
file to exist in both implementations and be named in both SKILLs; it requires
every invocation option (`-low -standard -high -light -full -pathworking`) in
both SKILLs; and it requires specific invariant sentences (e.g. "The Fulmen
brief is authoritative", "Your scope-limited task is your full authority",
"Scope drift is a defect", "Do not let a subagent's plan become the task") to be
present in **both** the codex and claude copies. When you change a role's prompt
or an invariant, change it in both places or `validate` fails.

The two role files for a given role carry the **same prose** but enforce
read-only / write scope through different platform mechanisms: codex via
`sandbox_mode = "read-only"`, claude via `disallowedTools: ["Edit", "Write", ...]`.
Keep the enforcement equivalent when editing either side.

### The six roles and the flow

Roles: `planner -> critic -> explorer -> worker -> verifier -> assembler`
(assembler is conditional). The parent session is always the orchestrator and
the source of truth; subagents return bounded, advisory outputs and never decide
user intent, broaden scope, or grant permissions. The canonical nine-step
sequence lives in `docs/architecture.md`; the platform-specific operator
instructions live in each `SKILL.md`. Read-only roles: planner, critic, explorer
(and verifier). Worker is the only role with a write scope.

### Pathworking layer

`docs/pathworking.md` is the conceptual (Tree of Life) model behind the flow and
the `-pathworking` invocation option. It is reference material — read it only
when working on `-pathworking` behavior or the symbolic framing, not for normal
edits.

## Editing checklist

1. Edit the role/skill file(s) — and the matching file in the **other**
   implementation if the change touches shared prose or invariants.
2. Run `./scripts/validate` until it prints `validate ok`.
3. YAML/TOML frontmatter values containing `: ` must be quoted — `validate`
   rejects unquoted frontmatter colons in both SKILLs and all claude agent
   files.
