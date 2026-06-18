---
name: fulmen
description: Orchestrate the Fulmen multi-agent harness: planner -> critic -> explorer -> worker -> verifier -> optional assembler. Use when the user invokes Fulmen or a task benefits from delegated subagent work.
---

# Fulmen

When the user invokes `/fulmen` with a task, or otherwise explicitly asks to use
Fulmen or subagents, you are the orchestrator. Run the sequence below. Do not
skip steps without stating why.

Fulmen uses these subagent roles: `planner`, `critic`, `explorer`, `worker`,
`verifier`, and `assembler`.

---

## Step 0 - Mode Decision

Assess the task complexity:

- **Lightweight**: well-understood task, narrow scope, no strategic ambiguity -
  jump to Lightweight Mode.
- **Full**: unclear scope, strategic choices, significant risk, or multiple
  moving parts - proceed with Step 1.

---

## Step 1 - Write the Fulmen Brief

Synthesize the user's task into a structured brief. Fill unknown fields with
`not specified` or a clearly labeled assumption. Ask one clarifying question only
when proceeding could change the intended outcome.

```text
## Fulmen Brief

**Goal**: [one sentence: what should be accomplished]
**Constraints**: [hard limits: what cannot change, what is off-limits]
**Definition of done**: [how to know the task is complete]
**Out of scope**: [explicit exclusions]
**File scope** (if known): [directories, files, or modules in play]
```

Share the brief as a concise progress update. Ask the user only if a missing
field creates a real risk of doing the wrong work; otherwise proceed.

---

## Step 2 - Spawn Planner

Invoke the `planner` agent. Pass the full Fulmen brief as its input. Wait only
when the next step is blocked on its result.

The planner returns a numbered plan with assumptions and resources listed.

---

## Step 3 - Spawn Critic

Invoke the `critic` agent after the planner completes. Pass both the Fulmen brief
and the planner's output.

The critic returns a structured problem list: risks, scope violations, missing
constraints, and unstated dependencies.

---

## Step 4 - Integrate

Resolve the plan against the critic's problem list and produce a working plan.
For each problem raised, mitigate, accept, or revise the plan before proceeding.

---

## Step 5 - Spawn Explorer

Distill from the working plan a single scoped question about the codebase or
system. Invoke `explorer` with that question only. Do not pass the full plan or
Fulmen brief.

The explorer returns structured findings: files, functions, dependencies, and
gaps.

---

## Step 6 - Spawn Worker

From the working plan and explorer findings, distill the scope-limited
implementation task. Invoke `worker` with:

- what to implement, specific and bounded
- which files are in scope
- a disjoint write scope: exact files or directories the worker owns and may edit
- definition of done from the Fulmen brief

Do not pass the full plan. Pass only the relevant slice.

---

## Step 7 - Verify

For non-trivial code changes, invoke `verifier` with:

- the Fulmen brief
- the worker completion report
- changed paths
- relevant explorer findings
- the behavior being claimed as complete

Tell the verifier it is read-only and must not edit files.

Skip verifier only for doc-only changes, purely mechanical edits, or tasks where
independent verification would add no useful evidence. State when it is skipped.

---

## Step 8 - Assemble (Conditional)

If worker and verifier output is complex, multi-part, or needs reconciliation,
invoke `assembler` with the relevant explorer, worker, and verifier outputs.

If the outputs are already clear and complete, skip assembler.

---

## Step 9 - Return

Review the final output against the Fulmen brief. Confirm definition of done,
scope boundaries, critic risks, and verifier findings before reporting back.

---

## Lightweight Mode

Use when scope is narrow and planning overhead is not warranted.

1. Write a short scope note.
2. Invoke `explorer` with a scoped question.
3. Invoke `worker` with the bounded task and explorer findings.
4. Invoke `verifier` for non-trivial code changes.
5. Review output against the scope note and report to the user.
