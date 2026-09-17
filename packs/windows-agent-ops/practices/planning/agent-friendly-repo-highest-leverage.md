---
{"id":"windows-agent-ops.planning.agent-friendly-repo-highest-leverage","title":"Highest-Leverage Changes for Agent-Friendly Repositories","stage":"planning","tech_stack":["agent-ops"],"applies_when":"a repository will be read and modified by AI coding agents and the team is choosing which structural investments to make first, or sequencing a remediation roadmap for an existing codebase","severity":"warn"}
---

## When to apply

Apply when scoping or triaging work whose goal is "make this codebase agent-friendly", deciding
between documentation, tooling, and refactoring investments, or prioritizing the backlog after an
agent-friendliness audit. Also apply when an agent repeatedly fails to build, verify, or locate code
in a repository it is expected to maintain.

## Guidance

Sequence the work by four levers, in this order, because each one amplifies the ones after it:

1. A single concise entry file at the repository root - AGENTS.md or the tool's recognized
   convention file - under roughly 200 lines: what the project is, how to run it, how to verify a
   change, and the constraints that are enforced. Long design history moves to linked documents,
   not into the entry file.
2. An executable verification loop: one documented command that runs the checks and returns a
   pass/fail verdict in seconds, cheap enough that an agent runs it after every edit. This is what
   lets an agent self-correct instead of asking a human to confirm each step.
3. A reference implementation: point at one existing good example - a canonical module, test, or
   config - rather than writing rules that describe how such code should look. Agents copy a
   concrete pattern far more reliably than they follow prose.
4. One source of truth per documented fact: a single authoritative document (or the code itself)
   per fact, with every other mention linking to it rather than restating it. Duplicated prose
   drifts apart silently, and an agent that trusts a stale copy makes confidently wrong changes.

Do not open with a large refactor. Its payoff only materializes once agents can already build,
verify, and navigate the tree cheaply.

## Why

The entry file is read and the verification command is run in nearly every agent session, so any
improvement to them compounds across all future tasks. A reference implementation replaces
specification with evidence that fits in a context window. A single documentation source of truth
keeps an agent from acting on a stale copy of a fact it had no way to know was stale. Classic
design quality - deep modules, narrow interfaces, greppable naming, deterministic checks - still
determines long-term health, but these four levers decide whether an agent can contribute at all
before that quality is reachable.

## Exceptions and boundaries

These are ordering heuristics, not a substitute for design: module boundaries, test coverage, and
dependency hygiene still govern long-term maintainability once the loop and entry file exist. A
repository with a machine-checked formatter and linter already satisfies much of the verification
lever. Generated or vendored trees may need explicit exclusions in the entry file instead of being
restructured, and an entry file that grows past its budget should be split by progressive
disclosure rather than trimmed at random.

## Example

Across a batch of repositories, the ones agents could change safely had a root entry file well
under 200 lines plus a "how to verify" command; the ones where agents failed had long or missing
entry documents and no single check command, so each session invented its own build steps. The
remediation order that followed was: write the entry file, add the verification script, fix
documentation drift, and only then start module cleanups - the cheap levers unblocked the rest.
