---
{"id":"windows-agent-ops.planning.dispatch-depth-limit","title":"Cap Agent Dispatch Depth at Two Levels and Forbid Nesting Below Grandchildren","stage":"planning","tech_stack":["agent-ops"],"applies_when":"a task is large enough to need several independent workstreams, or a dispatch instruction is being written for a subagent type that itself can dispatch further agents","severity":"warn"}
---

## When to apply

Apply when planning delegation for a task that splits into many independent units, and whenever
composing a dispatch instruction for an agent that has its own dispatch capability. Also apply when
reviewing a plan that reaches a third delegation level.

## Guidance

Allow at most two delegation levels: primary to subagent to grandchild. Nothing below a grandchild.

Only agent types that carry a dispatch tool can create a second level, and only when the primary
session explicitly authorizes it in the dispatch instruction. Grant that authorization only when the
task contains at least three independently executable units, and state the limits inline: maximum wave
size, background mode, and the rule that every grandchild instruction must contain an explicit line
saying it may not dispatch any further agents.

Agent types without a dispatch tool are natural leaves and need no such line, but the explicit
prohibition is still required for any grandchild that could pass work on. Never authorize dispatching
the general-purpose type from a grandchild, and keep at most three fan-out-authorized agents resident
at any time, counted against the same concurrency budget as other heavy agents.

## Why

Each delegation level multiplies context, memory, and cost, and a tree that grows one level deeper
than planned duplicates work and loses traceability. A grandchild cannot know it sits at the bottom of
the tree unless its instruction says so; the prohibition must be written down, not assumed. Bounding
depth also bounds how much of the execution tree the primary session can still verify and reason
about.

## Exceptions and boundaries

A single-unit task never needs a second level; give it one leaf agent. The depth budget is a ceiling,
not a target - most tasks fit in one level. If a plan genuinely needs three levels, restructure it into
sequential one-level dispatches instead of deepening the tree.

## Example

A migration splits into four independent workstreams. The primary session authorizes one
general-purpose agent to fan out coding and exploration subagents, caps the wave at ten, requires
background mode, and writes into every grandchild instruction a line stating it may not dispatch any
agents. The fan-out agent verifies the grandchild changes locally before reporting back to the primary
session, which runs its own review gate.
