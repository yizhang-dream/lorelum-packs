---
{"id":"windows-agent-ops.implementation.bypass-npm-sh-shims-when-conda-cygpath-shadows-msys","title":"Bypass POSIX npm Shims When a Conda cygpath Shadows MSYS","stage":"implementation","tech_stack":["windows","git-bash","msys"],"applies_when":"a globally installed npm CLI fails with MODULE_NOT_FOUND only under Git Bash while the .cmd or .ps1 launcher works from cmd.exe and PowerShell, especially with a conda distribution on PATH","severity":"warn"}
---

## When to apply

Apply when an npm-installed CLI exits with a module resolution error from Git Bash but runs correctly
in `cmd.exe` or PowerShell, and `type -a cygpath` shows more than one implementation.

## Guidance

Global npm packages ship three launchers: a POSIX `sh` shim, a `.cmd` shim, and a `.ps1` shim. The
`sh` shim locates its package by converting its own path with `cygpath -w`. A conda distribution
contributes `<conda-prefix>/Library/usr/bin/cygpath`, which usually precedes Git's `cygpath` on `PATH`
and resolves MSYS-style roots against the conda prefix - turning a path such as `/c/home/...` into
something like `C:\<conda-prefix>\Library\c\home\...`. The shim then points `node` at a directory that
does not contain the entry script, and the run dies with `MODULE_NOT_FOUND`.

The `.cmd` and `.ps1` shims use `%~dp0` and are unaffected, which is exactly why the failure is
bash-only and looks like a broken installation.

Bypass the shim instead of repairing it: invoke the package entry script with `node` directly, from a
PATH-front directory and behind an extensionless wrapper so every bash caller benefits.

```sh
#!/usr/bin/env sh
exec node "$HOME/.npm-global/node_modules/<pkg>/bin/<entry>.js" "$@"
```

More durable fixes: order `PATH` so the Git/MSYS `cygpath` precedes the conda one in bash sessions, or
reinstall the CLI into a standard npm prefix that avoids the mismatch.

## Why

The shim's base-directory detection is only as reliable as the `cygpath` it happens to find. Two
implementations with different mount tables cannot be mixed, and the shim has no way to tell which one
it got. Going straight to `node <entry>.js` removes that dependency entirely; delegating to the `.cmd`
shim is not an option under bash, since MSYS lookup does not resolve `.cmd` launchers by bare name.

## Exceptions and boundaries

A conda-exported `cygpath` is correct *inside* conda environments, so do not remove the conda prefix
from `PATH` globally to fix one tool; scope the change to bash sessions or to the wrapper. Packages
whose shims do not call `cygpath` need no wrapper. Re-check the wrapper after a major upgrade changes
the package layout under `node_modules`.

## Example

A global CLI prints `Cannot find module` under Git Bash while `where.exe <cli>` and the `.cmd` launcher
both work. `type -a cygpath` shows the conda copy first. An extensionless wrapper in `$HOME/bin` that
execs `node .../<pkg>/bin/<entry>.js` restores the command for all Git Bash sessions and sub-agents.
