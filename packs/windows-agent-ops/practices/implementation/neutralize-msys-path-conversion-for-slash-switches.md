---
{"id":"windows-agent-ops.implementation.neutralize-msys-path-conversion-for-slash-switches","title":"Neutralize MSYS Path Conversion for Slash-Prefixed Arguments","stage":"implementation","tech_stack":["windows","git-bash","msys"],"applies_when":"a native Windows executable called from Git Bash rejects a /switch argument or receives it as a C:/Program Files/Git/... path, or a PowerShell -Command string fails because bash expanded $_","severity":"warn"}
---

## When to apply

Apply whenever a Windows-native tool with slash options is driven from Git Bash: `taskkill /PID`,
`schtasks /create`, `sc.exe /query`, `wsl.exe ... /tmp/script.sh`, `robocopy /MIR`. The telltale
symptom is an argument that arrives as `C:/Program Files/Git/<token>`, or an "invalid argument" error
from a tool that is being given syntactically correct switches.

## Guidance

MSYS rewrites arguments that start with a single `/` and resemble POSIX paths into Windows paths before
the child process is created. Three countermeasures, in order of preference:

1. Double the leading slash for switch-only arguments: `taskkill //PID 1234 //F`. MSYS passes `//x`
   through as `/x`, so the native tool sees the switch it expects.
2. Disable conversion for one command with `MSYS_NO_PATHCONV=1 <cmd> ...` - use this when later
   arguments genuinely are POSIX paths that must reach the child verbatim.
3. When the native tool is PowerShell, wrap the entire command in single quotes so bash performs no
   expansion, and use double quotes only inside it:

```bash
powershell -NoProfile -Command 'Get-CimInstance Win32_Process | Where-Object { $_.CommandLine -like "*api*" }'
```

A double-quoted `-Command "... $_.CommandLine ..."` is expanded by bash into a file path before
PowerShell ever starts, producing a syntax error that has nothing to do with the script's logic.

## Why

MSYS has no per-tool knowledge of which switches exist, so it converts any single-slash token it cannot
distinguish from a path. Quoting does not prevent this - quoting is resolved by bash before conversion
happens - so only the `//` escape, the environment switch, or moving the invocation out of the bash
argument path removes the hazard.

## Exceptions and boundaries

Do not blanket-export `MSYS_NO_PATHCONV=1` in a shell profile: it also disables conversions that tools
legitimately need, and it hides rather than documents each case. Paths after the switch keep converting
normally under the `//` trick, which is what you usually want. When a command needs both a converted
path and a literal switch, prefer option 2 scoped to that single invocation.

## Example

`taskkill /PID 103108 /F` from Git Bash fails with "invalid argument" because the switch became a
Windows path. `taskkill //PID 103108 //F` succeeds, and `MSYS_NO_PATHCONV=1 taskkill /PID 103108 /F`
works equally well. The equivalent PowerShell one-liner must be single-quoted on the bash side.
