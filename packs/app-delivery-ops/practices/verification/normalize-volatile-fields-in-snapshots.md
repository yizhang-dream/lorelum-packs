---
{"id":"app-delivery-ops.verification.normalize-volatile-fields-in-snapshots","title":"Normalize Volatile Fields Before Comparing Snapshots","stage":"verification","tech_stack":["web","frontend"],"applies_when":"a golden-file or snapshot comparison fails intermittently with no functional change, because the captured output embeds timestamps, build ids, or other per-run values","severity":"warn"}
---

## When to apply

Apply when snapshot or golden-artifact comparisons are flaky across machines, days, or builds even
though the underlying behavior did not change. Also apply when designing a new capture pipeline,
before the first false positive teaches the team to ignore the check.

## Guidance

List the fields that legitimately vary per capture - build timestamps, cache-busting hashes, session
or request ids, absolute paths, ordering of unordered collections - and normalize exactly those
during capture: replace each with a fixed marker so the comparison still shows where the value sits.
Do the normalization in one shared function used by both capture and compare, and keep it minimal:
every normalized field is a class of regression you can no longer see.

Prefer fixing the source when a field is volatile only because it was embedded carelessly - make it
deterministic at generation time instead of masking it in the snapshot. Regenerate goldens only as a
deliberate, reviewed step, and keep the raw captured artifact available so a suspicious diff can be
inspected before normalization.

## Why

A volatile field makes the comparison fail for reasons unrelated to the change under test, and the
cheapest response - re-capturing blindly - trains everyone to accept snapshots without review, which
destroys the net's value. A small explicit normalization list keeps the signal while removing exactly
the known noise, and it documents what the team expects to vary.

## Exceptions and boundaries

Do not normalize a field just because it changed once; investigate first - a change in a field that is
normally stable is exactly the signal snapshots exist to catch. Values that must stay byte-identical
(for reproducible builds or distribution hashes) cannot be normalized away, only made deterministic.
Normalization happens before the diff, never after: do not post-process a failing diff to make it pass.

## Example

A page-snapshot suite compares server-rendered HTML fixtures. A build date embedded in the markup
makes every snapshot fail the day after capture. The fix strips the date field to a fixed marker in
the shared capture code, and a change that keeps the date but alters the surrounding markup still
fails as it should. The next render-affecting change re-captures the fixtures in the same commit.
