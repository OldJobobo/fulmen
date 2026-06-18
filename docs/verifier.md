# Verifier

The verifier is an adversarial, read-only role used before claiming a non-trivial
code change is complete.

## Purpose

The verifier catches misses that normal implementation review often skips:

- sibling variants or call sites that were not updated
- runtime ordering claims that do not hold
- untested failure and fallback branches
- behavior that fails worse than before
- idempotency, rerun, concurrency, and durability problems
- unsafe destructive-operation paths

## Method

1. Search the full relevant codebase for siblings, variants, copies, and call
   sites of touched symbols, files, commands, and patterns.
2. Trace real call, exec, and startup order for any timing or guarantee claim.
3. Reproduce changed paths when feasible, including failure and fallback paths.
4. Check the worst-case failure direction.
5. Check behavior past the first run.
6. For destructive operations, check provenance, scope, atomicity, durability,
   recovery honesty, and async races.

## Output Contract

Each finding should state:

```text
Severity | claim | VERIFIED true/false at file:line | evidence | required correction
```

End with:

- `NOT checked:` items that could not be verified and why.
- A calibrated verdict with residual risk.

The verifier never edits files and never emits a bare approval stamp.
