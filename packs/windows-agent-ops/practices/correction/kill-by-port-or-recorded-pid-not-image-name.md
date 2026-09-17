---
{"id":"windows-agent-ops.correction.kill-by-port-or-recorded-pid-not-image-name","title":"Kill Processes by Port or Recorded PID, Never by Image Name","stage":"correction","tech_stack":["windows","git-bash","msys"],"applies_when":"a stuck or orphaned process must be terminated and the shortest-looking remedy is an image-name kill such as taskkill /IM node.exe /F or taskkill /IM python.exe /F","severity":"warn"}
---

## When to apply

Apply before terminating any process on a shared Windows machine: restarting a dev server, cleaning up
a test listener, recovering a wedged port. Treat `taskkill /IM <name> /F` and
`Get-Process <name> | Stop-Process` as last resorts rather than shortcuts.

## Guidance

Identify the single process you own and kill exactly that one:

- By port, when the service holds one: `netstat -ano | findstr :<PORT>` gives the PID in the last
  column, then `taskkill //PID <pid> //T //F` (double slashes under Git Bash) or PowerShell
  `Stop-Process -Id <pid> -Force`.
- By PID recorded at start-up, which is the cheapest option: capture the PID when launching a
  long-lived process and store it with the task, so cleanup never has to search.

If a command-line match is unavoidable, constrain it by image name as well:

```powershell
Get-CimInstance Win32_Process | Where-Object { $_.Name -eq "python.exe" -and $_.CommandLine -match "my-service" } | ForEach-Object { Stop-Process -Id $_.ProcessId -Force }
```

Without the `$_.Name` condition, the match also hits every wrapper shell whose command line merely
mentions the keyword - including the shell that issued the kill - and the command returns an opaque
status such as 4294967295 (that is -1) with no useful error.

## Why

`node.exe` and `python.exe` are shared runtimes: an agent host, a test runner, a language server, and an
unrelated service can all live under the same image name. An image-name kill takes down every one of
them, and the damage is confusing rather than obvious - a front end keeps serving while its API dies,
which gets misdiagnosed as a network problem. Port-to-PID and recorded-PID targeting bound the blast
radius to the process you actually decided to stop.

## Exceptions and boundaries

An image name is acceptable only when it is provably unique to the workload, for example on a dedicated
build machine with no other consumers. When the PID is unknown and the port is unknown, enumerate
candidates, inspect each one's command line and start time, and confirm with the user before killing
anything. Never "clean up" by killing a runtime that other sessions may share.

## Example

A back-end listening on a test port is left running. `taskkill /IM python.exe /F` would also kill the
agent's own Python tooling and another service under the same interpreter. The port lookup yields one
PID, `taskkill //PID <pid> //T //F` removes exactly that tree, and the other services keep running.
