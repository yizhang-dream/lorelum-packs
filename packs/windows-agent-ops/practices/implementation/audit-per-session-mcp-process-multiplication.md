---
{"id":"windows-agent-ops.implementation.audit-per-session-mcp-process-multiplication","title":"Audit Per-Session MCP Process Multiplication Before Blaming Memory","stage":"implementation","tech_stack":["windows","agent-ops"],"applies_when":"an agent host becomes sluggish with several concurrent conversations and free memory or commit charge is nearly exhausted","severity":"warn"}
---

## When to apply

Apply when the host machine is slow with multiple conversations open, free memory headroom is small,
or sessions stall. Do not start from the assumption that conversation content is what got heavy -
first count what the sessions spawned.

## Guidance

Tool servers are typically started per session rather than shared. With N open sessions and M enabled
servers the process count grows toward N x M, and each server is usually fronted by its own launcher
process, which roughly doubles the count again. Plugin bundles that declare the same server inline
multiply it further.

Audit before changing anything:

1. Count processes by image name and list them together with their command lines.
2. Group by command line and count duplicates per server; the multiplicity should match the number of
   open sessions.
3. Compare the enabled server and plugin set with the observed set - extras usually mean a config
   change or an update re-enabled something.

Then cut, in this order:

- Check recent session transcripts for which servers were actually called. Anything with no recorded
  use is a candidate to disable.
- Disable servers and plugins by flipping their enable flags in the host config rather than
  uninstalling, so the change is reversible.
- Reclaim the running copies with kills matched on image name *and* command line. Never kill by image
  name alone - interactive runtimes and unrelated services share process names.
- Confirm the result with numbers: process count, free memory, free commit charge.

Configuration changes affect only newly started sessions, so exit all host instances before expecting
the new server set.

## Why

The multiplication is invisible in any single conversation - one session looks healthy - and the cost
lands on the machine as a whole, which makes it easy to misattribute to a heavy task or a leak. Counting
processes by command line turns a vague "the host is slow" into a concrete N x M number that points
straight at the configuration.

## Exceptions and boundaries

Do not disable a server that is in active use elsewhere just to recover memory; server-level disable
affects every future session equally. Killing processes is a stopgap - without the config change the
next sessions spawn them again. A single session with an unusually large tool set is a token problem
rather than a process problem and needs a different remedy.

## Example

An agent host with four conversations open reports only a few gigabytes of free memory. Counting
processes shows each of several servers present four times, one copy per session, plus launcher
parents. Disabling the unused servers in the config, killing the running copies by command line, and
restarting the host drops the process count and restores free memory.
