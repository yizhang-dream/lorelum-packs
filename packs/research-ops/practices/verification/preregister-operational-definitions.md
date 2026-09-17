---
{"id":"research-ops.verification.preregister-operational-definitions","title":"Pre-Register Operational Definitions","stage":"verification","tech_stack":["research","ml"],"applies_when":"a streaming or interactive system is about to be evaluated and the evaluation vocabulary - accuracy, latency, stability - is still being used informally in planning","severity":"warn"}
---

## When to apply

Apply before measuring any system whose outputs are revised over time or produced under
latency pressure - streaming transcription, live translation, incremental interactive agents
- and in particular before an A/B comparison of configurations.

## Guidance

Write operational definitions for every metric before running anything: what event counts as
final, which layer of context is in play, which timestamp anchors a latency, and what the
comparison baseline is. Separate quantities that look like one: decoding context, committed
output that no longer changes, and injected priors are three distinct things, and each
definition must state whether the metric includes it. Mark the boundary between the live layer
(stability and delay) and any offline refinement layer, and label the offline layer as
oracle-like refinement rather than ground truth.

Persist each session in two aligned formats - lossless raw media plus a structured event log
with per-line timing - so any configuration can be replayed offline against the same input.
Replay must schedule by absolute timestamps rather than by processing completion and must log
every event with source time and wall time, so latency and revision statistics are computed
from the log instead of felt from a live run. Freeze the API and the test matrix before
measuring, and report results as proxy metrics with the alignment detail attached.

## Why

Revision-based systems make informal vocabulary ambiguous in both directions: a "stability"
gain can come merely from committing later, and an "accuracy" claim can rest on a different
definition of final text. Without preregistered definitions a configuration comparison becomes
a debate about words, and any number can be made to look good by redefining the events.
Dual-format persistence is what makes the definitions testable: the same recording is
re-scored offline under a corrected definition, so fixing a definition never requires
recollecting data.

## Exceptions and boundaries

Recording and retaining raw media is a privacy-relevant act; keep retention bounded and
consent explicit. Proxy metrics must not be renamed to accuracy or error rate without a human
reference - agreement between two automatic decodes does not exclude a shared error, and a
proxy that is never validated against a reference stays a proxy.

## Example

Before running an A/B matrix, a streaming-transcription project wrote its operational
definitions into the project README: three kinds of context enumerated, revision rate
explicitly declared not equal to accuracy, and the offline refinement layer labeled proxy
rather than truth. Sessions were stored as aligned media plus event logs, so the preregistered
matrix and its questions could later be answered by offline replay alone.
