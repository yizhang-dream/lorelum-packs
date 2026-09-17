---
{"id":"windows-agent-ops.implementation.filter-tool-definitions-at-registration","title":"Filter Tool Definitions at Registration, Not at Permission Time","stage":"implementation","tech_stack":["agent-ops"],"applies_when":"tool definitions dominate the context window - most of the first-turn input tokens - or a permission-layer deny list is proposed as a way to shrink the prompt","severity":"warn"}
---

## When to apply

Apply when measuring why a session's context is nearly full before real conversation starts, and when
deciding how to remove tools the task does not need.

## Guidance

Tool definitions are injected into the prompt on every turn. When several plugins declare the same
remote server inline, definitions are duplicated wholesale: one endpoint can appear many times, and
duplicate copies can account for most of the injected tokens while the distinct tools are few.

Only filtering at registration removes tokens. The two layers are not interchangeable:

- Registration-time filtering - a startup flag that prunes entries before they are registered - keeps
  the definitions out of the prompt entirely. It matches exact tool names and has no wildcard support;
  a permission-style pattern such as `Bash(git *)` degrades to its leading token.
- Permission-layer deny lists are evaluated at call time. They block execution, but the definitions
  are still injected, so they save nothing.

Measure rather than assume: sum the injected tool names and the first-turn input token count from
session transcripts, apply the change, and re-measure. Attribute the delta to registration filtering
only if nothing else changed at the same time.

Prefer disabling the whole plugin or server when none of its tools were called - check the transcripts
for actual invocations first. Per-tool flags are the fallback for surgical cases, and the flag list
gets unwieldy quickly, so pair it with a wrapper that keeps the invocation readable.

## Why

The prompt cost is paid on every request, including turns where no tool runs, so registration-time
filtering improves the whole session rather than a single call. Permission-layer filtering is a safety
mechanism, not a budgeting mechanism; treating the two as equivalent leads to configurations that look
restrictive while the context stays full.

## Exceptions and boundaries

Measure in an interactive session: headless runs may fail to connect some remote servers at all, so
savings observed there prove nothing about normal use. Tool-name matching is exact and case-sensitive,
so verify the names against the recorded tool list before pruning. Configuration and flag changes take
effect for newly started sessions, and a workspace-level config cannot remove a server contributed by
an installed plugin - disable the plugin instead.

## Example

A first-turn transcript shows hundreds of tool definitions and a six-figure input token count, with one
server appearing a dozen times because several plugins declare it inline. Adding the duplicate groups
to a startup filter list drops the first-turn count sharply; an equivalent permission-layer deny list
changes nothing in the same measurement.
