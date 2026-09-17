---
{"id":"toolchain-deploy-ops.planning.preflight-bridges-before-dispatch","title":"Preflight Runtime Bridges and Skills Before Dispatching Agents","stage":"planning","tech_stack":["agent-ops","toolchain"],"applies_when":"a work plan is about to be handed to parallel sub-agents whose tasks depend on a live MCP bridge, an engine hook, a service endpoint, or domain skills being available at runtime","severity":"warn"}
---

## When to apply

Apply at the start of any wave of parallel agent work whose workers depend on a runtime bridge (an MCP
server, a debug hook, a local service) or on domain knowledge shipped as skills. Run the preflight
before the first worker is dispatched, not after one of them has failed.

## Guidance

Three checks, in order. First, probe the bridge with its cheapest read-only call and treat a failure as
blocking: report it and ask for an environment restart rather than routing around the bridge or letting
workers proceed without it. Second, load the domain skills the work depends on; skills are the
construction knowledge source and a worker will not necessarily discover them on its own. Third, size
the wave and dispatch it with exclusive file ownership - one worker per file set, background mode, no
shared write targets.

State bridge availability in the worker instructions. If the sub-agents have no access to the bridge
tools at all, give them the CLI equivalent and the exact command line, and tell them what to do when the
tool is missing instead of letting them fall back silently.

## Why

A dead or unregistered bridge does not fail loudly at dispatch time. Workers start, make no progress on
the parts that need it, and return confident-sounding reports built on nothing - the failure surfaces
hours later as "the work is done but nothing works". Asking the owner for a restart is far cheaper than
a wave of silently degraded results. Exclusive ownership matters for the same reason: parallel workers
editing one file produce overwrite races that read as flaky work rather than as a dispatch error.

## Exceptions and boundaries

Do not restart shared infrastructure other sessions depend on merely to satisfy a preflight; request
the restart and wait for it. Skills are not a substitute for a bridge when the work is stateful (for
example, driving a running engine) - they cover static knowledge only. If the bridge comes back mid-wave,
finish the current wave before adding bridge-dependent tasks, so results do not mix provenances.

## Example

A plan called for several agents to implement engine features in one repository. Instead of dispatching
immediately, the session first called the engine bridge's version endpoint and confirmed it answered,
and loaded the physics and headless-workflow skills - then dispatched workers in waves, each owning a
disjoint set of files. In a later session the bridge was down; the correct move was to ask the user to
restart the host and wait, because agents without the bridge can write code but cannot verify it against
the running engine, and their reports would have been unverifiable.
