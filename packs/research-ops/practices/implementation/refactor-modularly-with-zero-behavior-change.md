---
{"id":"research-ops.implementation.refactor-modularly-with-zero-behavior-change","title":"Refactor Modularly With Zero Behavior Change","stage":"implementation","tech_stack":["research","agent-ops"],"applies_when":"splitting a large flat package into subpackages while scripts, notebooks, entry points, and external callers still import the old module paths","severity":"warn"}
---

## When to apply

Apply when a research codebase is reorganized - flat files into subpackages, modules moved or renamed -
and other parts of the system (training entry points, cloud job scripts, notebooks, tests) import paths
that are about to disappear. Also apply before approving any refactor whose selling point is "pure
structure, no behavior change".

## Guidance

Keep every old import path working through a compatibility shim, and make shim usage go down rather
than up:

- Leaf modules: alias the whole module into `sys.modules` under the old name so identity, attributes,
  and private names are preserved.
- Packages that must avoid import cycles: use a lazy facade resolving attribute reads, writes, and
  deletes against the real implementation with no cached copy.
- Choose alias over re-export-star with a decisive test: names with a leading underscore are reachable
  only through the old path, so any consumer of one forces a full alias.
- When the facade sits below the import system, guard against the parent-link `setattr` the importer
  itself performs; a naive delegate there breaks half-initialized cycle contracts.
- Freeze public CLI surfaces byte for byte: the canonical `python -m <pkg>.<entry>` invocation, its
  arguments, and its output must stay unchanged, because external runners use both module and file mode.
- Keep path boilerplate in one module, and preserve any "import implies chdir" contract that other
  scripts silently depend on.
- Record every legacy path with its current reference count in a registry, and ratchet it in the gate:
  exceeding the baseline fails, so migration can only shrink and new code cannot adopt dead paths.
- Prove zero behavior change per batch: full test suite, linter, and import-layer contract checks green,
  plus a snapshot replay of a real workload comparing pre- and post-refactor trees field by field.
- Land each batch as independently revertible commits, and have a different reviewer clear each one.

## Why

Large refactors drift silently: unit tests pass while an external script, a serialized artifact, or an
import-order-sensitive path breaks. The shim keeps old consumers alive during migration, and the
ratchet keeps the migration monotone; without the ratchet, new code re-adopts old paths and the debt
never retires. Snapshot replay catches drift that tests do not model, and per-batch review keeps the
blast radius small enough to bisect.

## Exceptions and boundaries

Do not shim across process boundaries: a deployment script pinned to old module names must be updated
in the same change, not aliased. Delete an alias once its usage count reaches zero. Behavior
comparisons require pinned inputs and artifacts; when a change intentionally alters a serialized format
(for example, a hash that includes fully qualified class names), declare digests non-comparable across
the boundary and forbid cross-run comparison rather than forcing byte equality.

## Example

A flat package of 87 files was regrouped into 13 subpackages. Legacy imports kept working through
aliases and lazy facades, the CLI entry point stayed byte-identical, and each batch passed the full
suite plus a same-command replay of a real workload whose result lines matched field by field. The new
shim registry then caught a parallel branch reintroducing two legacy paths the same day.
