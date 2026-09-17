---
{"id":"windows-agent-ops.implementation.register-scheduled-tasks-via-powershell-xml","title":"Register Windows Scheduled Tasks from PowerShell with a UTF-16 XML Definition","stage":"implementation","tech_stack":["windows","git-bash","msys"],"applies_when":"an agent needs a recurring or catch-up Windows task and direct schtasks /create from Git Bash is mangled, or the scheduled-task cmdlets refuse registration with access denied","severity":"warn"}
---

## When to apply

Apply when creating or updating a Windows scheduled task from an agent session. Both common entry
points fail in ways that look like permission problems but actually come from the invocation route or
from one element in the definition.

## Guidance

Register from PowerShell, passing a UTF-16 XML definition file to the classic tool:

```powershell
schtasks /create /tn <TaskName> /xml <path>\task.xml /f
```

Write the XML as UTF-16 - the encoding `schtasks /xml` expects - and keep the definition minimal.
Two dead ends to skip:

- Running `schtasks /create` directly from Git Bash: MSYS rewrites `/create`, `/tn`, `/xml`, and `/f`
  into Windows paths, so the tool receives a nonsense command line.
- Using the newer cmdlet API: registration is rejected with 0x80070005 even when an explicit
  principal is supplied, and adding a logon trigger to the XML is refused the same way.

Get catch-up behavior from the start-when-available setting instead of a logon trigger: if the machine
was off or the agent host was down at the scheduled moment, the task runs at the next opportunity.
Unless a later first fire is deliberate, set the trigger start boundary to the current day - a future
boundary means no run and no catch-up until that date.

When the task starts work through a wrapper that returns immediately, mark the wrapper to wait on its
child (wait-on-return). Otherwise the task is treated as finished at once and the whole process tree is
reaped, killing work still in progress.

## Why

The classic tool plus an XML file keeps every parameter out of the shell, so MSYS has nothing to
rewrite, and it bypasses the cmdlet path whose access checks reject an otherwise valid definition.
Encoding matters because `/xml` reads the file with a fixed expectation: a mismatched encoding turns
into a parse error that reads like a syntax mistake in the definition.

## Exceptions and boundaries

This path needs an elevated or at least interactive desktop session; a non-interactive agent context
may be unable to register tasks at all, in which case ship the XML plus a one-line registration
command for a human to run. The start-boundary rule is documented separately; check it before changing
a schedule. Do not fall back to bypassing the security model when access is denied - recheck the
invocation route first.

## Example

A nightly model-routing switch must run at fixed hours. Direct `schtasks` calls from Git Bash fail with
path-shaped switches, and the cmdlet returns access denied. Writing the definition as UTF-16 XML and
running `schtasks /create /tn <TaskName> /xml <path>\task.xml /f` from PowerShell registers it, and the
start-when-available setting covers nights when the machine was asleep.
