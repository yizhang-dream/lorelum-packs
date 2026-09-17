---
{"id":"research-ops.implementation.record-fixed-schema-run-metadata","title":"Record Run Metadata in a Fixed Schema","stage":"implementation","tech_stack":["research","ml"],"applies_when":"long or repeated training, simulation, or evaluation runs produce checkpoints and progress logs, and results from different runs must later be compared, filtered, or joined","severity":"warn"}
---

## When to apply

Apply when a training or simulation loop is being written or migrated, when checkpoints are saved
without context, or when run descriptions currently live only in notes, chat, or filenames. The
trigger is any workflow where two runs of the same command can produce artifacts that a reader
cannot distinguish afterwards.

## Guidance

Define one small metadata schema and let code write it, never a human after the fact:

- Keep the field set minimal and stable: run identity, configuration summary or config hash, data or
  state version, code revision, start time, seed, and status. Add fields only with defaults; never
  rename or repurpose an existing field. Bump a schema version field when meaning changes.
- Emit a sidecar metadata file next to every checkpoint in the same write. The checkpoint filename
  stays mechanical; semantics live in the sidecar, so a checkpoint can be understood after the run
  process is gone.
- Log progress in a machine-readable fixed order - one CSV or JSONL row per step or evaluation,
  written by the training loop itself - so a run killed midway still leaves a parseable tail.
- Register each run in the experiment ledger or index using the same identity key as the metadata,
  so no run exists only as prose.

Before starting a run, check that the metadata writer is actually wired into the checkpoint path. A
schema that is only documented but not emitted is worth nothing.

## Why

Hand-collected notes drift, omit the fields that matter later, and cannot be joined across runs.
A fixed schema makes comparisons mechanical: filter by status, group by config hash, order by start
time. Sidecar metadata survives context loss, and fixed-order progress rows keep partial runs
usable, which matters most exactly when a run dies early and its evidence is still wanted.

## Exceptions and boundaries

Throwaway probes that cannot influence any decision may skip full metadata, but the moment a run's
output might be reported, it needs at least identity, config hash, code revision, and status. Do not
put large payloads - metrics tables, parameter dumps - inside the metadata file; reference them.
A schema migration that changes existing field meanings requires versioning, not a silent update of
the writer. Keep secrets and credentials out of metadata fields.

## Example

A trainer writes a metadata sidecar with four frozen fields - run identity, configuration hash,
dataset version, and code revision - alongside each checkpoint. Progress rows are emitted by the
loop into a fixed-column log rather than collected by hand, and the ledger entry is generated from
the same identity key. A later run dies at mid-training; its partial log still parses cleanly and
its last checkpoint is identifiable from the sidecar, so the evidence is not lost.
