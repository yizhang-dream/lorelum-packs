---
{"id":"toolchain-deploy-ops.testing.never-git-stash-in-shared-worktrees","title":"Never Use git stash in a Shared Working Tree","stage":"testing","tech_stack":["agent-ops","toolchain"],"applies_when":"multiple agents share one working tree and any of them plans to use git stash to temporarily remove changes while verifying a failure path or a clean-tree build","severity":"warn"}
---

## When to apply

Apply when several agents work in the same checkout at the same time and one of them needs to test a
failure path, a clean-tree build, or the pre-change behavior. Apply whenever a verification step
proposes stashing to obtain a temporary state.

## Guidance

Proof of a failure path must not depend on tree-wide state. Instead of stashing, temporarily edit the
specific file, run the one check, and restore that file immediately - or better, inject the failure in
process through a fixture, an environment flag, or a copy of the file under a temporary name.

Forbid whole-tree operations in a shared checkout: stash and pop, checkout of all paths, hard resets,
and clean of untracked files. If a whole-tree operation is genuinely required, serialize it and
announce the window; the durable fix is a private worktree or branch per agent, which makes the
operation invisible to everyone else.

## Why

Stash is a global operation on shared mutable state with no ownership boundary: it removes every
uncommitted change, not only the caller's. In a shared tree the window is visible to sibling agents as
a sudden, unexplained regression - a gate that passed a moment earlier fails mid-run - so the failure
gets attributed to the wrong change and time is spent debugging a ghost. Restoration can also conflict
with work landed during the window, turning a verification trick into real data loss.

## Exceptions and boundaries

Stashing is safe in a private worktree or branch that no other agent reads. Read-only verification that
never mutates the tree needs no stash at all. Temporary edit-and-restore is equally unsafe if the
window is long, so keep it to a single command and restore before running any other check, and never
combine it with a parallel gate run.

## Example

One agent edits a validation branch to confirm an error is detected and uses stash to also check the
success path. During that window a sibling agent's full gate run fails on documents the first agent had
touched, and the failure is misreported as a regression from an unrelated change. The correct
technique: edit the single file, run the one check, restore the file immediately, and leave every other
path in the tree untouched.
