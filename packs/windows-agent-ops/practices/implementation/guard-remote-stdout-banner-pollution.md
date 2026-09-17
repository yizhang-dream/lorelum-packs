---
{"id":"windows-agent-ops.implementation.guard-remote-stdout-banner-pollution","title":"Guard Against Banner and Profile Noise in Remote Command Output","stage":"implementation","tech_stack":["git-bash","agent-ops"],"applies_when":"an agent parses stdout from a remote login command and assumes the first lines are data, or reads a value by line position","severity":"warn"}
---

## When to apply

Apply whenever output is consumed programmatically from a login session on a remote host: ssh
one-liners, remote diagnostics, log fetches, version probes. Any interactive login path can prepend
text that the parsing code never expects.

## Guidance

A login shell emits more than the command's output. Scan-code or message-of-the-day banners and
profile warnings - version-manager notices, package-manager config advisories - can run to dozens of
lines before the first real line of data. Never index output by absolute line number, and never assume
the only interesting line is the last one.

Use one of these patterns instead:

- Two-step file handoff: the first remote command writes its result to a temp file; a second command
  `cat`s that file. The payload then arrives without any login preamble.
- Marked section: bracket the payload with unique begin/end markers and extract only what lies between
  them (`sed -n '/BEGIN/,/END/p'`).
- Noise reduction: non-interactive shell flags and disabling the remote banner help, but profile
  output belongs to the remote machine and cannot be fully controlled from the client side.

Capture the remote exit status separately, before piping into a local filter, since a pipe makes `$?`
report the local tool's status and a failed remote command then looks successful.

## Why

The banner is produced by the login path, not by your command, so it is invisible in the command you
wrote and stable only until the remote operator changes a profile file. Treating stdout as pure data
makes parsing correct on one host and silently wrong on the next; a file handoff or an explicit marker
makes the boundary between transport noise and payload something the command itself states.

## Exceptions and boundaries

A remote command that legitimately prints structured output first (a JSON line before diagnostics) can
still be parsed directly - the hazard is ordering assumptions, not the extra lines themselves. Do not
suppress stderr globally to hide banner noise: error text from the command shares that stream, and
muting it converts real failures into empty results. On hosts you control, disabling the banner once
is cleaner than parsing around it.

## Example

A probe that reads the first line of a remote version command reports a scan-code banner string
instead of a version number. Writing the version command's output to a temp file on the remote host
and `cat`-ing that file in a second invocation returns the value alone, with the exit status captured
before any local pipe.
