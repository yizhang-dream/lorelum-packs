---
{"id":"toolchain-deploy-ops.implementation.rename-fields-atomically-across-consumers","title":"Rename Fields Atomically Across Every Consumer","stage":"implementation","tech_stack":["web","toolchain"],"applies_when":"a data field, key, or flag is being renamed in a codebase whose consumers include templates, generated data, or rendering paths that string-keyed lookups and type checkers do not cover","severity":"warn"}
---

## When to apply

Apply to any rename of a field that is read by more than one rendering path - especially when consumers
include frozen or hand-maintained templates, generated data files, JSON fixtures, snapshots, or prose
documentation. Apply before starting the rename, not after the first consumer has been updated.

## Guidance

Enumerate consumers first: grep the old identifier across the whole repository, including template
strings, generated data, fixtures, scripts and docs, and keep the list open until the grep is empty.
Then change producer and every consumer in one batch, together with any derived or generated artifact
that embeds the name.

Prefer renames that fail loudly. Do not keep a silent compatibility read of the old name unless an
externally published format requires it, because a fallback quietly hides consumers the rename missed.
Add a rendering-level regression assertion that would have caught the miss - for example, that a fresh
consumer of the data shows the neutral state while an existing one preserves its state.

## Why

Renames are cross-cutting contract changes, but string-keyed reads - template attributes, dynamic
property access, generated data keys - are invisible to compilers and to tests that exercise only the
path the author remembered. A partially applied rename therefore ships a real behavioral regression
behind a green test suite: every read of the old key yields undefined and the UI falls back to a
default that looks plausible to anyone who did not see the previous rendering.

## Exceptions and boundaries

Keep a documented, deprecated alias only for formats others already consume - on-disk artifacts,
published API responses - and record when it can be removed. Do not rename and change semantics in the
same batch: rename first, then change behavior, so any regression has a single candidate cause.
Generated data must be regenerated in the same batch as the rename, never hand-patched.

## Example

A publication flag is renamed from one word to another. The first pass updates the component that
renders items, plus its tests, and everything passes. But a frozen root template still reads the old
name for the course-level list, so all entries render as "coming soon". A second pass updates the
template, the generated navigation data and the docs, and adds a regression check on the fresh-visitor
state that fails if the rename is ever partial again.
