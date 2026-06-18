# Fulmen Roles

## Planner

The planner is read-only. It receives the full brief and returns an ordered plan
with assumptions and required resources. It does not implement.

## Critic

The critic is read-only. It receives the brief and planner output, then returns a
problem list only: risks, scope violations, missing constraints, and unstated
dependencies. It does not suggest fixes.

## Explorer

The explorer is read-only. It receives one scoped question about the codebase or
system and returns files, symbols, dependencies, and gaps relevant to that
question.

## Worker

The worker has write access. It receives only a bounded implementation task,
explicit file scope, ownership boundaries, and definition of done. It makes the
minimum necessary change and reports what changed.

## Verifier

The verifier is read-only. It receives the brief, worker report, changed paths,
relevant explorer findings, and claimed behavior. It checks whether the change
actually holds across the broader feature surface and runtime path.

## Assembler

The assembler may write only its own final output. It receives explorer, worker,
and verifier outputs when they need reconciliation. It adds no new facts or
recommendations.
