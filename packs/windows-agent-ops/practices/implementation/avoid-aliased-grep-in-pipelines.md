---
{"id":"windows-agent-ops.implementation.avoid-aliased-grep-in-pipelines","title":"Avoid grep in Pipelines When a Shell Profile Aliases It","stage":"implementation","tech_stack":["windows","git-bash","msys"],"applies_when":"grep -E or a multi-flag grep call inside a Git Bash pipeline fails with a matcher conflict or behaves unlike the documented tool","severity":"warn"}
---

## When to apply

Apply when `grep` with extended or combined flags fails from an interactive Git Bash session, or when
a pipeline that worked inside a script fails when typed into a shell. The error usually concerns
conflicting matchers rather than a missing file or bad pattern.

## Guidance

A shell profile can alias or wrap `grep` with extra default flags - a color flag, an ignore-case flag,
a matcher-mode flag. Folding that default together with the flag you passed produces a conflict the
real binary never sees, and the wrapper's behavior drifts from the documented tool.

Confirm the cause before rewriting the pipeline: `type grep` and `alias grep` show whether a wrapper
shadows the binary, and prefixing a single call with `command` or an absolute path bypasses the alias.

For pipelines, prefer filters without profile overrides:

```bash
sed -n 's/.*"name":"\([^"]*\)".*/\1/p'
awk '/^pattern/ { print $2 }'
cut -d, -f3
```

When the shape is complex, write the raw output to a file and parse it with a real scripting runtime
(`node -e`, `python -c`) instead of chaining fragile text filters. Test the replacement against the
same input the failing pipeline saw, not against a hand-made sample.

## Why

The alias lives in the interactive shell's startup files, so the same command behaves differently
between a typed session and a script - exactly the environment split an agent operates in. Removing
grep from the pipeline avoids the wrapper entirely and also removes a common source of exit-code
masking, since filters return their own status rather than the producer's.

## Exceptions and boundaries

`command grep` or the absolute binary path is the right fix when grep itself is the best tool and the
alias is only cosmetic - do not rewrite a working pattern into a worse one just to avoid the command.
Aliases are per-shell: a non-interactive script that never loads the profile is unaffected, so a bug
that reproduces only interactively points here, while one that reproduces everywhere does not. Fixing
the profile is out of scope for a one-off pipeline; note the wrapper and move on.

## Example

An agent pipeline calls `grep -E` to extract identifiers and fails with conflicting matchers, while the
identical line works when run from a script. `alias grep` shows a color flag injected by the profile.
Replacing the filter with `sed -n 's/.../\1/p'` makes the pipeline work in both environments.
