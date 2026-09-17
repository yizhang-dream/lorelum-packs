---
{"id":"windows-agent-ops.implementation.git-bash-bare-command-lookup-misses-cmd-shims","title":"Git Bash Bare-Command Lookup Does Not Find .cmd Shims","stage":"implementation","tech_stack":["windows","git-bash","msys"],"applies_when":"a CLI resolves from cmd.exe, PowerShell, or where.exe but every Git Bash session reports command not found, and the install directory contains only a .cmd (or .ps1) launcher","severity":"warn"}
---

## When to apply

Apply when a freshly installed or updated CLI is reported as missing from Git Bash even though
`where.exe <name>` finds it and the same bare command works in `cmd.exe` or PowerShell. Look at what
the installer actually wrote before concluding the tool is not installed.

## Guidance

MSYS command lookup, like POSIX `execvp`, tries the bare name and an explicit `.exe` suffix. It does
not apply the full Windows `PATHEXT` list, so `.cmd`, `.bat`, and `.ps1` launchers are invisible to a
bare invocation. The tool is installed; only the resolver disagrees.

Restore a bare-command name by placing an extensionless `sh` wrapper in a directory that is early on
`PATH` (for example `$HOME/bin`) and letting it exec the official shim:

```sh
#!/usr/bin/env sh
exec "$HOME/AppData/Roaming/npm/<name>.cmd" "$@"
```

Delegating to the installed shim - rather than to a versioned program path - keeps the wrapper working
across upgrades, and `exec` keeps it transparent: same stdio, same exit status, no extra process.

## Why

Windows launchers rely on `PATHEXT` so that a bare `name` resolves to `name.cmd`. MSYS emulates POSIX
semantics on top of Windows and only widens the search with `.exe`, so the two lookup rules disagree
for exactly the suffix that package managers emit. The disagreement is invisible from cmd.exe and
PowerShell, which is why the failure gets misread as "the agent host cannot see an installed tool"
rather than as a suffix-resolution gap.

## Exceptions and boundaries

If `where.exe` also fails, this is a PATH visibility problem, not a `.cmd` problem - check whether the
registry PATH gained the directory after the shell took its environment snapshot. Do not fix the
symptom by pinning a versioned binary path in the wrapper, and do not settle for a shell alias: other
agents, sub-agents, and scripts that call the bare name will still fail.

## Example

An installer writes `lore.cmd` into an npm global prefix. `where.exe lore` succeeds and PowerShell runs
`lore` normally, but every Git Bash session and sub-agent gets `command not found`. Adding an
extensionless `$HOME/bin/lore` that execs `lore.cmd` fixes all callers without touching the install.
