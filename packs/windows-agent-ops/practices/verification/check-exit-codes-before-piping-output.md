---
{"id":"windows-agent-ops.verification.check-exit-codes-before-piping-output","title":"Capture Exit Codes Before Piping Output","stage":"verification","tech_stack":["git-bash","msys"],"applies_when":"a verification command pipes a tool's output into tail, head, grep, or sed and success is then judged from the pipeline exit status","severity":"warn"}
---

## When to apply

Apply to every acceptance or CI-style command whose verdict is read from the shell exit status. The
moment output is piped, the status you observe belongs to the last element of the pipeline, not to the
tool under test.

## Guidance

Redirect the tool's combined output to a file first, capture `$?` on the next statement, then inspect
the file with any filter you like:

```bash
npx tsc --noEmit > .verify/tsc.log 2>&1; ec=$?
[ "$ec" -eq 0 ] || echo "tsc exited $ec"
tail -30 .verify/tsc.log
```

The two statements must be separate: `cmd | tail -30` followed by `echo $?` reports the filter's
status, and a build with dozens of type errors will look like a pass. In bash, `set -o pipefail` makes
the pipeline surface the rightmost non-zero status, which is a useful complement but still not a
substitute - it does not tell you which stage failed or preserve the producer's status when a later
stage also fails.

## Why

A pipeline's exit status is defined by its last command. `tail`, `head`, and `grep` exit 0 whenever they
ran at all, so they mask the producer's failure. Agent acceptance loops that trust `$?` after a pipe
therefore mark broken builds as verified - the failure is not detectable from the log alone, since the
log may even contain the error text that was ignored.

## Exceptions and boundaries

When the filtered output is only for human reading and the real gate is elsewhere (a CI job, a test
runner that sets its own status), capturing the status is unnecessary. If another tool later reads the
log file, write it to a repository-local path instead of `/tmp`: Git Bash's `/tmp` maps to its own
temporary directory, while a Node process resolves `/tmp/...` as `C:\tmp\...` and reports ENOENT.

## Example

`npx tsc --noEmit 2>&1 | tail -30; echo $?` prints 0 while the log is full of type errors. Splitting
it into `> log 2>&1; ec=$?` followed by `tail -30 log` shows both the true exit code and the errors,
so the verification record reflects reality instead of the filter's cheerful status.
