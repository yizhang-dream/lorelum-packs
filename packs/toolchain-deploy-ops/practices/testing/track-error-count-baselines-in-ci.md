---
{"id":"toolchain-deploy-ops.testing.track-error-count-baselines-in-ci","title":"Track Error-Count Baselines Instead of Demanding Clean Runs","stage":"testing","tech_stack":["toolchain","web"],"applies_when":"CI is asked to execute notebooks, tutorials, or training scripts that intentionally contain exercises or unsolved placeholders, so a strict assertion gate can never pass","severity":"warn"}
---

## When to apply

Apply when wiring continuous integration for artifacts that cannot run clean by design: exercise
notebooks with learner placeholders, tutorial scripts with deliberate unresolved cells, or any corpus
whose failures are part of the teaching material. Also apply when a scheduled anti-rotation job starts
failing on a clean tree and the temptation is to disable it.

## Guidance

Record a baseline of expected error counts per artifact in a committed data file. Run the artifacts
with errors tolerated, then fail the job when a count deviates, an artifact is missing from the report,
or a new failure appears outside the baseline. Require any baseline change to land in the same change
set as the content change that explains it, so a baseline bump cannot be slipped in alone.

Keep secondary checks informational. Version-drift or freshness checks should report findings without
failing the build, so the failure signal stays reserved for real regressions. Add only CI setup steps
that fit the repository: dependency caching keyed on a requirements file fails outright in a repo that
has none.

## Why

When clean execution is unachievable, a strict gate is disabled or ignored within weeks, and the
project loses the signal entirely. The useful signal for such corpora is change: a count that drifts,
a new failure class, an artifact that stopped executing at all. A baseline turns an all-or-nothing gate
into a ratchet that still catches regressions while letting intentional exercises exist, and making the
baseline a reviewed artifact keeps the "intentional failure" claim honest.

## Exceptions and boundaries

Artifacts expected to run clean keep strict assertion gates; baselines are only for documented,
inherent failures. A baseline change with no corresponding content change is a smell and should be
reviewed as a possible regression. Do not let informational checks grow into obstacles: the moment a
drift report blocks merges it has become a gate, and it needs an owner and a threshold.

## Example

Seven teaching notebooks execute in CI with errors allowed, and a baseline file records the expected
error-cell count for each. A change that fixes one exercise cell must update that notebook's count in
the same change set; a count that changes without a content change fails the job. A separate check
reports dependency version drift but never fails, and the language setup step uses no package cache
because the repository has no requirements file to key it on.
