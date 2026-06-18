# Fulmen Architecture

Fulmen is an orchestration pattern, not one large agent prompt. The parent agent
keeps ownership of the user conversation and delegates bounded work to role
agents.

## Sequence

1. The parent writes a brief with goal, constraints, definition of done, file
   scope, and exclusions.
2. `planner` turns the brief into an ordered approach.
3. `critic` reviews the plan and reports only risks, scope issues, missing
   constraints, and hidden dependencies.
4. The parent integrates those findings into a working plan.
5. `explorer` answers a scoped discovery question.
6. `worker` performs a bounded implementation task.
7. `verifier` checks the completed change for breadth, runtime order, failure
   direction, repeatability, and destructive-operation safety.
8. `assembler` runs only when multiple outputs need a single coherent summary.
9. The parent performs the final review and reports gaps or completion.

## Operating Rules

- Delegate bounded sidecar work only when it materially advances the task.
- Use platform-supported budget controls conservatively: preserve inherited
  defaults when uncertain or when risk, ambiguity, or implementation complexity
  is high.
- Keep planner and critic read-only.
- Keep explorer read-only.
- Give worker an explicit write scope.
- Give verifier completed claims and changed paths, not an open-ended task.
- Use assembler only for assembly, not new analysis.
- The parent remains responsible for final correctness.
