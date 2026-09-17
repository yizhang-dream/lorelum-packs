---
{"id":"toolchain-deploy-ops.testing.require-all-green-before-commit","title":"Require All Green and a Zero-Diff Rebuild Before Committing","stage":"testing","tech_stack":["toolchain"],"applies_when":"a change is about to be committed to a repository that has an executable test suite and a deterministic build step, especially after a refactor, a dependency change, or a scripted edit across many files","severity":"warn"}
---

## When to apply

Apply at every commit gate of a project that produces build output and has tests, and especially after
refactors, dependency changes, or batch edits that touch many files at once.

## Guidance

Two gates, both mandatory. Tests: run the full suite, not the subset that merely looks related, and
commit only on a fully green run - a skipped suite is an untested change. Build determinism: rebuild
the artifact from the sources in the tree and require a zero diff against the artifact already present
(or against the previous build for generated output). Byte-identical output proves the artifact is
reproducible from source and that nobody edited it by hand.

Keep the gates fast enough to run every time. When the runner can select suites from changed paths, use
it, but fall back to the full set for changes that touch shared code, and never let the selector
silently classify a new directory or a moved file as unrelated - verify the mapping when the layout
changes. Write the exact commands into the repository so the next person runs the same gates.

## Why

A commit is a claim about the state of the tree, and both gates defend that claim. A partial test run
misses the suite that would have caught a shared-code regression; an unchecked artifact makes the
shipped output differ from the sources with no diff for anyone to review. Determinism is also the
cheapest detector of hidden inputs: a rebuild that differs proves some input is not captured by the
build, whether a timestamp, an absolute path, or a hand edit.

## Exceptions and boundaries

Builds that embed timestamps, absolute paths, or generated identifiers cannot meet a literal zero-diff
gate; normalize those fields or hash a canonical projection of the output instead of dropping the gate.
An emergency commit that skips the suite must be labeled as such and repaired before the next release.
A zero-diff rebuild proves reproducibility, not correctness - the test gate carries that side.

## Example

A project's release discipline reduced to one line: the full suite must be green, and rebuilding the
single-file artifact must leave the build output byte-identical. A refactor commit that looked
test-neutral still failed the build gate because the bundler's output changed, so the change was
reviewed instead of shipping as an invisible artifact edit. When selective suite running was added
later, the same gate stayed in place, with the selector applying only to the test step and the full
suite kept as the fallback for shared code.
