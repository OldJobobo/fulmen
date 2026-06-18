# Setup

Fulmen is installed by linking this repo into the live Codex and Claude config
directories.

## Commands

```bash
./scripts/fulmen-live status
./scripts/fulmen-live install
./scripts/fulmen-live doctor
./scripts/validate
```

## Backup Behavior

If a live target already exists and is not a symlink into this repo, `install`
moves it to:

```text
~/.local/share/fulmen/backups/<timestamp>/
```

Then it creates the symlink. Repo-owned symlinks are updated in place.

## Uninstall

```bash
./scripts/fulmen-live uninstall
```

Uninstall removes only symlinks that point into this repo. It does not delete
backups and does not remove unmanaged files.

## Optional Claude Hooks

The verifier protocol can also be enforced through Claude hooks, but this repo
does not enable hooks by default. Add hook wiring only after verifying the local
Claude hook configuration and desired stop behavior.
