---
{"id":"windows-agent-ops.implementation.verify-cli-visibility-after-path-installs","title":"Verify CLI Visibility After PATH-Modifying Installs","stage":"implementation","tech_stack":["windows","git-bash","agent-ops"],"applies_when":"a newly installed CLI works in a fresh terminal but is reported missing (command not found) by an already-running agent host, IDE, or long-lived shell session, and the agent is about to conclude the tool is not installed","severity":"warn"}
---

## When to apply

Apply when a tool or CLI was installed (often with an installer that appends its shim directory to
the user registry PATH), and a session that predates the install cannot find the bare command. Also
apply before diagnosing any "CLI not installed" claim that comes from a long-running host process.

## Guidance

Do not trust a single shell's PATH as proof of installation state. Compare three things: the user
registry PATH (`[Environment]::GetEnvironmentVariable('Path','User')` in PowerShell), the current
session's `$PATH`, and the install directory itself. If the registry has the entry but the session
does not, the running process tree inherited an environment snapshot taken before the install;
Windows processes capture their environment at creation and never re-read the registry.

The fix is to restart the host process tree so new children inherit the updated environment. For
immediate work inside the stale session, invoke the binary by absolute path instead of re-installing
or "repairing" the tool.

## Why

Windows propagates environment-variable changes only to processes launched after the change from a
parent that has the new value (e.g. a fresh Explorer shell). Long-lived agent hosts and their child
shells keep the old snapshot indefinitely, so `command not found` from such a session says nothing
about whether the tool exists - only that the session cannot see it.

## Exceptions and boundaries

A registry PATH entry that is missing from a *newly launched* terminal is a different failure (entry
not actually written, wrong scope, or shell profile overriding PATH) and needs its own diagnosis.
Do not restart shared infrastructure (databases, remote sessions) merely to refresh PATH; use
absolute paths there instead.

## Example

An agent host started at 00:24 reports `lore: command not found` after lore was installed at 00:28,
leading a task report to claim "CLI not installed". The registry PATH contains the shim directory
and the binary runs fine by absolute path, so the correct diagnosis is a stale environment snapshot;
restarting the host restores bare-command resolution.
