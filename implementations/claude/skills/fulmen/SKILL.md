---
name: fulmen
description: "Orchestrate the Fulmen multi-agent harness: planner -> critic -> explorer -> worker -> verifier -> optional assembler. Use when the user invokes Fulmen or a task benefits from delegated subagent work."
---

# Fulmen

When the user invokes `/fulmen` with a task, or otherwise explicitly asks to use
Fulmen or subagents, you are the orchestrator. Run the sequence below. Do not
skip steps without stating why.

Fulmen uses these subagent roles: `planner`, `critic`, `explorer`, `worker`,
`verifier`, and `assembler`.

Before spawning, identify the immediate critical-path task you should do locally
and which sidecar tasks can run in parallel. Delegate only bounded sidecar work
that materially advances the run. Do not delegate work whose result is required
before you can take the next local step.

The parent session remains the source of truth. Subagents provide bounded
outputs; they do not decide user intent, broaden scope, or authorize work.

---

## Claude Subagent Tooling

Spawn subagents with the `Agent` tool. Set `subagent_type` to `planner`,
`critic`, `explorer`, `worker`, `verifier`, or `assembler`.

- An `Agent` call returns the subagent's final output as the tool result.
  Achieve parallelism by issuing multiple independent `Agent` calls in a single
  message; run a step sequentially only when it is blocked on a prior result.
- Keep `planner` -> `critic` serial: the critic must receive the planner output.
- Claude subagents start from a fresh context. Inherited parent context is not
  passed unless you include it in the subagent prompt, so pass each agent only
  the bounded slice it needs.
- Claude subagents are one-shot: they complete, return a single result, and end.
  There is no persistent agent thread to close, so spawn fresh bounded subagents
  for each Fulmen run rather than carrying agents between runs.
- Reuse a live subagent (via `SendMessage` to a still-running agent) only for a
  same-run follow-up that genuinely depends on that agent's existing context.
- Fulmen defines no Claude-specific reasoning-effort or budget override. Use the
  platform's normal agent invocation behavior unless a supported Claude setting
  is explicitly available in the current environment.

---

## Invocation Options

Fulmen accepts compact skill-level options in the user prompt. These are not
native CLI arguments; interpret them as orchestration preferences.

- Budget: `-low`, `-standard`, `-high`
- Flow: `-light`, `-full`, `-pathworking`

If options conflict, prefer safety: `-high` beats `-low`, `-full` beats
`-light`, and `-pathworking` combines with either flow mode. Mention unknown
options in the Fulmen brief and ignore them unless they create real ambiguity.

- `-low`: optimize token use. Keep more sphere work in the parent session and
  spawn only subagents that materially reduce risk or context pollution.
- `-standard`: default Fulmen behavior. Choose Lightweight or Full mode from
  task complexity.
- `-high`: prioritize quality over token reduction. Keep verifier for
  non-trivial changes.
- `-light`: use Lightweight Mode unless the task grows enough to require Full.
- `-full`: use the full planner -> critic -> explorer -> worker -> verifier ->
  optional assembler flow.
- `-pathworking`: perform a compact parent-side Malkuth -> Kether diagnosis
  before normal execution. Do not spawn extra agents only for the symbolic
  framing.

For Tree of Life background, read `pathworking.md` (bundled beside this skill)
only when the user asks
for `-pathworking`, asks a conceptual Fulmen question, or the task depends on
that model.

---

## Translation Discipline

Each Fulmen handoff must preserve the user's request instead of paraphrasing it
into a different task.

- Keep the user's exact request visible in the Fulmen brief.
- Convert intent into subagent prompts only by narrowing scope, never by adding
  new goals or permissions.
- Treat subagent output as advisory evidence. The parent must reject any
  subagent suggestion, edit, or conclusion that exceeds the brief.
- If planner, critic, explorer, worker, or verifier output conflicts with the
  user's request, the Fulmen brief, or the declared file/write scope, the parent
  resolves the conflict before continuing.
- Do not let a subagent's plan become the task. The task remains the user's
  request plus the parent-approved brief.

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

**User request**: [quote or near-verbatim restatement of the user's actual ask]
**Goal**: [one sentence: what should be accomplished]
**Constraints**: [hard limits: what cannot change, what is off-limits]
**Definition of done**: [how to know the task is complete]
**Out of scope**: [explicit exclusions]
**File scope** (if known): [directories, files, or modules in play]
**Delegation rules**: [what subagents may and may not decide]
```

Share the brief as a concise progress update. Ask the user only if a missing
field creates a real risk of doing the wrong work; otherwise proceed.

---

## Step 2 - Spawn Planner

Invoke the `planner` agent. Pass the full Fulmen brief as its input. Wait only
when the next step is blocked on its result.

The planner returns a numbered plan with assumptions and resources listed.

After receiving the planner output, remove or ignore any plan step that changes
the user request, expands scope, or assumes permission not present in the brief.

---

## Step 3 - Spawn Critic

Invoke the `critic` agent after the planner completes. Pass both the Fulmen brief
and the planner's output. Wait for it to complete only when integration is
blocked on its result.

The critic returns a structured problem list: risks, scope violations, missing
constraints, and unstated dependencies.

Ask the critic to include translation drift: any place where the plan changes
the user request, weakens constraints, or implies unapproved work.

---

## Step 4 - Integrate

Resolve the plan against the critic's problem list. For each problem raised:

- **Risk**: note it as a known risk; decide whether to mitigate or accept.
- **Scope violation**: remove or revise the offending step.
- **Missing constraint**: add it to the working plan.
- **Unstated dependency**: name it explicitly in the working plan.

Produce a working plan. This drives the rest of the sequence.

Before delegating further, write a parent-approved working plan. This plan must
name the allowed write scope, rejected planner/critic items, and any accepted
risks. Do not pass unresolved ambiguity to worker.

---

## Step 5 - Spawn Explorer

Distill from the working plan a single scoped question about the codebase or
system. Invoke `explorer` with that question only. Do not pass the full plan or
Fulmen brief.

Wait for it to complete only when the worker prompt or local critical-path work
is blocked on its result. The explorer returns structured findings: files,
functions, dependencies, and gaps.

If the explorer answers a broader question than requested, use only the findings
that match the scoped question.

---

## Step 6 - Spawn Worker

From the working plan and explorer findings, distill the scope-limited
implementation task. Invoke `worker` with:

- what to implement, specific and bounded
- which files are in scope
- a disjoint write scope: exact files or directories the worker owns and may edit
- definition of done from the Fulmen brief

Tell the worker to make its edits in the working tree, report changed paths, and
describe the behavioral change. Tell it it is not alone in the codebase, must not
revert edits made by others, and must adjust to existing concurrent changes. Do
not pass the full plan. Pass only the relevant slice. Explicitly tell the worker
that the slice is its entire authority; it must stop and report if the task
requires files, behavior, or decisions outside that slice.

Wait for the worker to complete only when review or integration is blocked on its
result. Review the returned changes before integrating, refining, or reporting
them. Reject or revert worker output that exceeds the approved slice.

---

## Step 7 - Verify

For non-trivial code changes, invoke `verifier` with:

- the Fulmen brief
- the worker completion report
- changed paths
- relevant explorer findings
- the behavior being claimed as complete

Tell the verifier it must not edit files; Bash is permitted only for read-only
checks. It should check blast radius, sibling variants, runtime order, failure
paths, repeat runs, and destructive-operation safety where relevant.
Ask it to verify against the original user request and Fulmen brief, not just
the worker report.

Skip verifier only for doc-only changes, purely mechanical edits, or tasks where
independent verification would add no useful evidence. State when it is skipped.

---

## Step 8 - Assemble (Conditional)

If worker and verifier output is complex, multi-part, or needs reconciliation,
invoke `assembler` with the relevant explorer, worker, and verifier outputs. Tell
it to assemble only from the provided outputs, add no new content, and return a
single coherent deliverable.

If the outputs are already clear and complete, skip assembler.

---

## Step 9 - Return

Review the final output against the Fulmen brief:

- Does it still answer the user's actual request?
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

1. Invoke `explorer` with a scoped question derived from the scope note.
2. Invoke `worker` with the scope-limited task, disjoint write scope, file
   ownership, and explorer findings.
3. Invoke `verifier` for non-trivial code changes.
4. Review output against the scope note and report to the user.
