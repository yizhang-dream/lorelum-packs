---
{"id":"windows-agent-ops.planning.model-tiered-dispatch","title":"Route Implementation and Monitoring Work Off the Expensive Primary Model","stage":"planning","tech_stack":["agent-ops"],"applies_when":"an agent session running on an expensive primary model is about to spend its own turns writing code, fixing bugs, or watching long-running jobs, instead of delegating that work to cheap fast subagents","severity":"warn"}
---

## When to apply

Apply when a session is backed by an expensive primary model and its task includes implementation work
(writing or editing files, fixing bugs, adding tests, refactoring) or long-running observation
(watching experiments, remote jobs, log tailing, hardware health checks). Also apply at planning time,
when deciding who executes each part of a multi-step task.

## Guidance

Reserve the expensive primary session for planning, review, decisions, and answers to the user. Route
execution work to cheap fast subagents, which are individually slower per call but can run many at
once and cost little.

Every dispatch instruction must be self-contained and carry four things:

1. Objective - the single outcome the subagent must produce.
2. The files, directories, or commands involved.
3. Checkable acceptance criteria - which command to run, which behavior to observe, or what the diff
   should look like. These become the reviewer checklist, so vague criteria produce unverifiable work.
4. Known pitfalls - environment traps, prior failures, and conventions the subagent must follow.

Keep dispatch instructions short and templated; shorter instructions are cheaper to write and easier
to fan out.

## Why

Expensive-model capacity is the scarce resource in the session; cheap subagent capacity is abundant
and parallel. Spending primary turns on mechanical execution trades the one resource that can plan and
judge for a resource that cannot. Acceptance criteria are load-bearing: without them the subagent
cannot self-check, the reviewer has nothing objective to compare against, and rework is forced.

## Exceptions and boundaries

One-line or few-line edits, pure questions, and tasks with no file operations are cheaper to do
directly than to dispatch. Keep work in the primary session when it depends on the full conversation
context and cannot be restated as a self-contained instruction. Do not dispatch work whose acceptance
criteria cannot yet be articulated - define them first, then dispatch.

## Example

A session on an expensive model needs a path fix, a test update, and a six-hour training run watched.
It writes the plan and acceptance criteria itself, dispatches the code change and the test update to
cheap coding subagents in parallel and a monitoring subagent to the training run, then reviews the
returned diffs and reports to the user, spending its own turns only on judgment.
