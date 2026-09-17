---
{"id":"windows-agent-ops.implementation.repair-missing-windowsapps-user-path","title":"Repair a Dropped WindowsApps Alias Directory in the User PATH","stage":"implementation","tech_stack":["windows","git-bash","msys"],"applies_when":"an app-execution-alias command such as pwsh or winget reports command not found in freshly launched terminals even though the package is still installed","severity":"warn"}
---

## When to apply

Apply when a command delivered as a Store/MSIX app-execution alias stops resolving, and the failure
appears in *new* shells rather than only in long-running sessions. The same pattern applies to any
tool whose launcher is an alias file rather than a real executable under `Program Files`.

## Guidance

Store-style packages do not install into `Program Files`. They register aliases inside
`%LOCALAPPDATA%\Microsoft\WindowsApps`, and that single directory is what the user PATH must contain.
When the entry is dropped - by cleanup scripts, PATH editors, or an unrelated uninstaller rewriting the
user PATH - every alias in it stops resolving at once while the packages remain installed.

Diagnose in this order:

1. Read the user PATH from the registry, not from the current shell:
   `[Environment]::GetEnvironmentVariable('Path','User')`.
2. List the alias directory and confirm the launcher file for the missing command is present.
3. If the registry entry is missing, add the directory back, then open a fresh terminal.

Do not conclude "not installed" from a missing command alone. MSIX packages have no uninstall entry
under the usual per-user registry keys, so the ordinary install-check commands cannot see them.
Confirm installation through the alias file or the package list, and upgrade through the package
manager rather than a downloaded installer.

## Why

One directory holds aliases for many unrelated tools, so a single dropped PATH entry looks like a mass
uninstall and invites reinstalls that cannot fix anything. Separating "package gone" from "alias
directory unreachable" first keeps the repair to one registry write.

## Exceptions and boundaries

A missing command in an already running session is a different failure: processes inherit the
environment snapshot taken when they started, so a session older than the PATH edit cannot see the
restored entry - restart that process tree instead of reinstalling. Do not move a per-user alias
directory into the system PATH, and do not change system-wide console encoding or locale settings to
chase unrelated output garbling while repairing PATH.

## Example

`pwsh` starts failing with command not found in new Git Bash windows. The user registry PATH no longer
contains `%LOCALAPPDATA%\Microsoft\WindowsApps`, while the directory and its launcher files are still
on disk. Appending the directory to the user PATH and opening a new terminal restores the command; no
reinstall is needed.
