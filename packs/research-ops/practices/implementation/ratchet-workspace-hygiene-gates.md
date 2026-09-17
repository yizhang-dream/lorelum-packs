---
{"id":"research-ops.implementation.ratchet-workspace-hygiene-gates","title":"Ratchet Workspace Hygiene Gates","stage":"implementation","tech_stack":["agent-ops","research"],"applies_when":"a repo used by long-running agents accumulates tracked junk, oversized files, or tracked files that .gitignore also matches, and a full cleanup is not scheduled","severity":"warn"}
---

## When to apply

Apply when a repository is routinely used by agents or long-running tooling and starts accumulating
tracked scratch output, junk-named files, or files that are committed while a `.gitignore` pattern also
matches them. Also apply when writing hygiene rules into a project charter and wanting them to hold.

## Guidance

Ship hygiene as its own gate module plus a baseline file, wired into the fast check target; do not
inline the logic into an existing gate script. Define three ratchets and two hard checks:

- Ratchet A: number of tracked files at the repository root.
- Ratchet B: tracked files whose names match junk patterns (scratch, temp, test output).
- Ratchet C: tracked files that a `.gitignore` pattern also matches.
- Hard check 1: files above a size threshold that are not on an allowlist.
- Hard check 2: known scratch directories that exist but are not ignored.

Semantics matter more than the list: the baseline records the current state and the gate fails only on
growth - block new offenders, do not demand cleanup of existing ones. The baseline may only move down
as debt is paid; a version that allowed raising the baseline in either direction was rejected in review
as a loophole. Allow exceptions only for the hard limits (for example a documented large fixture),
never for the ratchets.

Treat "tracked yet ignored" files as repository content, not configuration: removing them from the
index changes what is committed, so absorb the current count into the baseline, report the finding, and
wait for owner sign-off before running any destructive command. Evidence and forensic artifacts are
usually irreplaceable - report them, do not delete them.

## Why

Rules written in a charter do not constrain tools; only a machine gate does. Left alone, a single tool
state directory reached thousands of files and hundreds of megabytes of agent junk, because cleanup was
never anyone's task. A ratchet converts the same rule into something that fails the fast check the next
time someone adds a junk file, while still shipping today without a big-bang cleanup.

## Exceptions and boundaries

Retire the gate when the repository no longer hosts long-running agent tooling. Do not use the baseline
as a place to hide planned debt: if a ratchet trips, delete or relocate the file. Files under version
control that ignore rules match usually indicate an earlier mistake in the ignore file; check both
directions before deciding which side to fix. A hard-check allowlist entry needs a stated reason next
to it, or it becomes the next loophole.

## Example

A project added a hygiene module plus a baseline JSON as an item in its fast check. It blocked the first
new root-level scratch file a week later, while the measured backlog - tens of thousands of files in one
tool state directory - stayed recorded as baseline debt for later batch cleanup instead of becoming a
stop-the-world task.
