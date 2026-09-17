---
{"id":"windows-agent-ops.implementation.check-path-resolution-before-trusting-system-commands","title":"Verify PATH Resolution Before Trusting a System Command","stage":"implementation","tech_stack":["windows","git-bash","msys"],"applies_when":"a familiar system command such as cmd, node, or python fails with an error that belongs to an unrelated package, or behaves differently from the same command in another shell","severity":"warn"}
---

## When to apply

Apply when a supposedly built-in Windows command returns a confusing error - a module-not-found message,
an unexpected argument parse, a failure from a package you never invoked - or when two shells disagree
about what a command name means.

## Guidance

Resolve the name before trusting it. In Git Bash, `type -a <cmd>` lists every candidate in resolution
order, so the first line is what will actually run. In PowerShell, `Get-Command <cmd> -All` shows the
same picture.

A package manager can install an executable that shares a name with a system tool. A globally installed
package literally named `cmd` produces a shim such as `~/.npm-global/cmd`, and because user package
prefixes typically precede `C:\Windows\System32` on `PATH`, every caller - including tools that shell
out to `cmd /c ...` internally - gets the shim and dies with an error from that unrelated package.

Remedies, in order: call the system binary by absolute path (`/c/Windows/System32/cmd.exe`); better,
skip the intermediate shell and invoke the real target executable directly; long term, repair or remove
the shadowing shim, or order `PATH` so system directories precede user package prefixes.

## Why

`PATH` is a first-match-wins search across heterogeneous directories, and an installation into a user
prefix silently shadows a system tool for every child process. The error text is produced by the
shadowing tool, so it names a package and a failure mode that have nothing to do with the command the
caller asked for - which sends diagnosis in the wrong direction until resolution order is checked.

## Exceptions and boundaries

If `type -a` reports a single system path, the resolution is unambiguous and the bug is the command's
own. Shell functions and aliases shadow ahead of `PATH`; `type -a` reports them first, and a wrapper or
alias can be intentional, so inspect before removing. Prefer an absolute path over renaming the
offending package, and revisit resolution order again after any tool installs into a user prefix.

## Example

`cmd //c start.bat` from Git Bash fails with a module-not-found error from an uninstalled package.
`type -a cmd` shows a shim under the npm global directory first and `System32\cmd.exe` second. Calling
`/c/Windows/System32/cmd.exe //c ...` works, and invoking the target executable directly avoids `cmd`
altogether.
