---
name: fulmen
description: Orchestrate the Fulmen multi-agent harness: planner -> critic -> explorer -> worker -> verifier -> optional assembler. Use when the user explicitly invokes Fulmen or asks for delegated subagent work on a task.
---

# Fulmen

When the user invokes `$fulmen` with a task, uses `/fulmen` as an alias, or
otherwise explicitly asks to use Fulmen or subagents, you are the orchestrator.
Run the sequence below. Do not skip steps without stating why.

Fulmen uses these subagent roles: `planner`, `critic`, `explorer`, `worker`,
`verifier`, and `assembler`.

Before spawning, identify the immediate critical-path task you should do locally
and which sidecar tasks can run in parallel. Delegate only bounded sidecar work
that materially advances the run. Do not delegate work whose result is required
before you can take the next local step.

---

## Codex Subagent Tooling

Spawn subagents with `multi_agent_v1.spawn_agent`. Set `agent_type` to
`planner`, `critic`, `explorer`, `worker`, `verifier`, or `assembler`.

- Omit model overrides unless the task specifically needs one.
- Keep `fork_context: false` unless inherited context is explicitly useful.
- Wait with `multi_agent_v1.wait_agent` only when the next step is blocked on
  that agent's result. While agents run, continue non-overlapping local work
  when useful.
- Keep planner -> critic serial: the critic must receive the planner output.

---

## Step 0 - Mode Decision

Assess the task complexity:

- **Lightweight**: well-understood task, narrow scope, no strategic ambiguity -
  jump to Lightweight Mode at the bottom.
- **Full**: anything with unclear scope, strategic choices, significant risk, or
  multiple moving parts - proceed with Step 1.

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

Use `multi_agent_v1.spawn_agent` with `agent_type: "planner"`. Pass the full
Fulmen brief as its input. Wait for it to complete only when the critic cannot
be spawned without its result.

The planner returns a numbered plan with assumptions and resources listed.

---

## Step 3 - Spawn Critic

Use `multi_agent_v1.spawn_agent` with `agent_type: "critic"` after the planner
completes. Pass both the Fulmen brief and the planner's output. Wait for it to
complete only when integration is blocked on its result.

The critic returns a structured problem list: risks, scope violations, missing
constraints, and unstated dependencies.

---

## Step 4 - Integrate

Resolve the plan against the critic's problem list. For each problem raised:

- **Risk**: note it as a known risk; decide whether to mitigate or accept.
- **Scope violation**: remove or revise the offending step.
- **Missing constraint**: add it to the working plan.
- **Unstated dependency**: name it explicitly in the working plan.

Produce a working plan. This drives the rest of the sequence.

---

## Step 5 - Spawn Explorer

Distill from the working plan a single scoped question about the codebase or
system. Use `multi_agent_v1.spawn_agent` with `agent_type: "explorer"` and that
question only. Do not pass the full plan or Fulmen brief.

Wait for it to complete only when the worker prompt or local critical-path work
is blocked on its result. The explorer returns structured findings: files,
functions, dependencies, and gaps.

---

## Step 6 - Spawn Worker

From the working plan and explorer findings, distill the scope-limited
implementation task. Use `multi_agent_v1.spawn_agent` with
`agent_type: "worker"` and:

- what to implement, specific and bounded
- which files are in scope
- a disjoint write scope: exact files or directories the worker owns and may edit
- definition of done from the Fulmen brief

Tell the worker to edit directly in its forked workspace, report changed paths,
and describe the behavioral change. Tell it it is not alone in the codebase,
must not revert edits made by others, and must adjust to existing concurrent
changes. Do not pass the full plan. Pass only the relevant slice.

Wait for the worker to complete only when review or integration is blocked on
its result. Review the returned changes before integrating, refining, or
reporting them.

---

## Step 7 - Verify

For non-trivial code changes, use `multi_agent_v1.spawn_agent` with
`agent_type: "verifier"`. Pass:

- the Fulmen brief
- the worker completion report
- changed paths
- relevant explorer findings
- the behavior being claimed as complete

Tell the verifier it is read-only and must not edit files. It should check
blast radius, sibling variants, runtime order, failure paths, repeat runs, and
destructive-operation safety where relevant.

Skip verifier only for doc-only changes, purely mechanical edits, or tasks where
independent verification would add no useful evidence. State when it is skipped.

---

## Step 8 - Assemble (Conditional)

If worker and verifier output is complex, multi-part, or needs reconciliation,
use `multi_agent_v1.spawn_agent` with `agent_type: "assembler"` and pass the
relevant explorer, worker, and verifier outputs. Tell it to assemble only from
the provided outputs, add no new content, and return a single coherent
deliverable.

If the outputs are already clear and complete, skip assembler.

---

## Step 9 - Return

Review the final output against the Fulmen brief:

- Does it meet the definition of done?
- Were any out-of-scope changes made?
- Were the critic's risks addressed, mitigated, or explicitly accepted?
- Were verifier findings resolved or explicitly reported?

Report the review to the user. Flag any gaps. This closes the run.

---

## Lightweight Mode

Use when scope is narrow and planning overhead is not warranted.

Write a brief scope note:

```text
**Goal**: [what to accomplish]
**File scope**: [relevant files or directories]
**Definition of done**: [what done looks like]
```

Then:

1. Use `multi_agent_v1.spawn_agent` with `agent_type: "explorer"` and a scoped
   question derived from the scope note.
2. Use `multi_agent_v1.spawn_agent` with `agent_type: "worker"` and the
   scope-limited task, disjoint write scope, file ownership, and explorer
   findings.
3. Use `multi_agent_v1.spawn_agent` with `agent_type: "verifier"` for
   non-trivial code changes.
4. Review output against the scope note and report to the user.
