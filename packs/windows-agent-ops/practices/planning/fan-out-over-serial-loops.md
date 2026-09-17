---
{"id":"windows-agent-ops.planning.fan-out-over-serial-loops","title":"Fan Out Independent Units Instead of Queuing a Batch Through One Agent","stage":"planning","tech_stack":["agent-ops"],"applies_when":"a task contains several independent objects (files, directories, commands, candidates, datasets) and the plan would hand the whole batch to a single subagent to process in a loop","severity":"warn"}
---

## When to apply

Apply whenever the work item is plural: N files to migrate, N directories to scan, N commands to run,
N candidate approaches to evaluate. Also apply when a draft dispatch instruction contains sequencing
words such as one by one, then, next, or and also, or when a single prompt contains several
read-modify-verify cycles.

## Guidance

List the full batch first, one object per line, then send one dispatch call per object in the same
message so they run concurrently. Give every call the same template and change only the object
parameter. In systems that support it, send large batches in background mode and start preparing the
next batch instead of waiting for results.

Two packing rules survive the fan-out requirement:

- Same file, one agent. Never let two concurrent agents edit the same file; keep all edits to a given
  file inside a single dispatch to prevent write conflicts.
- Strongly coupled units may share an agent: several functions in one module, or edits that must be
  designed together.

For large batches, split into waves and cap the number of heavy agents per wave. Harvest a wave before
launching the next one.

## Why

A serial loop through one agent turns a batch of parallel work into wall-clock time multiplied by the
number of objects and makes the whole batch depend on a single context window. Parallel dispatch
matches the concurrency most agent systems already have. A shared template makes writing N dispatches
cheaper than writing one oversized prompt, so fan-out is also the lower-effort option.

## Exceptions and boundaries

True dependencies must stay ordered: if object B needs the result of object A, fan out within each
stage, not across stages. Units that must be designed in one context belong in one dispatch even if
they touch different files. Keep per-wave concurrency bounded; oversized waves exhaust memory and
context and degrade every agent in the wave.

## Example

Twenty configuration files need the same migration. The planner lists all twenty, sends twenty
dispatches in one message with identical instructions and one changed path each, in two waves of ten,
then harvests results per wave. The same work queued through a single agent would take many times
longer and risk losing earlier edits to context pressure.
