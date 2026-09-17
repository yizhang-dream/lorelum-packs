---
{"id":"research-ops.implementation.coordinate-parallel-agents-through-git","title":"Coordinate Parallel Agent Lines Through Git, Not Assumptions","stage":"implementation","tech_stack":["agent-ops","research"],"applies_when":"two or more agent sessions share one working tree and one execution host, and either line may launch jobs, edit documents, or commit while the other is working","severity":"warn"}
---

## When to apply

Apply whenever more than one line of work may be writing to the same checkout or launching on the same
shared machine - including the case where the other line is invisible to you and you only suspect it
because output directories or processes look unexpected.

## Guidance

Treat the shared repository's commit history as the coordination mechanism; nothing else is shared.
Before launching any job on shared compute, inspect the target output directory and the process table
for the script name, excluding the search process itself. Before recording results or committing, pull
fast-forward and check status.

Commit only the paths you changed. Do not use a full-tree staging command: a working tree with
another line's uncommitted work will silently absorb it. Use a path-scoped commit that does not touch
the index entries staged by others, and add new files to the index first so the pathspec matches.
When the tree contains another line's leftover fix, compare it against the committed description and
the actual code; if it is consistent, adopt it into your own themed commit with a note, rather than
discarding it or claiming it as a fresh discovery.

If an edit refuses with a modified-since-read error, the other line is writing that file: re-read,
then edit. Never write blind. Resolve interpretive disagreements with bucketed measurements rather
than seniority - a coarse aggregate can rank configurations that a per-cell breakdown separates.

## Why

Two lines that do not know about each other still collide: two launches can write the same output
directory, making one run's data unattributable, and two commits can interleave so that a full-tree
add sweeps half-finished work into an unrelated commit. Path-scoped commits and pre-launch directory
and process checks close both failure modes for a few seconds of cost each, and git history is the
only record both lines are guaranteed to read.

## Exceptions and boundaries

When a working tree belongs to one line exclusively, the pre-launch checks can be lightened and
path-scoped commits are optional. Never overwrite or revert another line's staged content, and never
force-push a shared branch. Process checks on a remote host must exclude the checking shell's own
command line, or they will report false liveness.

## Example

Two lines launch probe jobs on the same host with the same output path; one run's artifacts have to be
discarded because the directory was written by two processes. Later, a commit must be recorded while
an unrelated refactor batch sits uncommitted in the tree: the operator commits with an explicit list of
only its own paths, notes in the commit message that other documents carry the parallel batch's
changes, and leaves the batch untouched.
