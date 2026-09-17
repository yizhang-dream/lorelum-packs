---
{"id":"windows-agent-ops.verification.report-only-in-run-measured-data","title":"Report Only In-Run Measured Comparisons in Evaluations","stage":"verification","tech_stack":["research"],"applies_when":"an agent is assembling a benchmark or comparison report for models, tools, or configurations and is about to fill a table cell with a score taken from a public leaderboard, a vendor model card, or a web search result","severity":"warn"}
---

## When to apply

Apply to any deliverable that compares models, tools, or configurations on measured dimensions:
benchmark write-ups, evaluation reports, head-to-head tool comparisons, ablation tables.
Apply before adding any externally sourced number to a results table, and when closing out a run
whose tables contain a row the agent did not execute itself.

## Guidance

Make each report self-contained in its own run. Every score in the comparison tables must come from
the commands and corpus executed in this session, with one row designated as the baseline - normally
the configuration the requester already owns or distrusts least. Keep the run artifacts: exact
commands, harness configuration, dataset or corpus revision, and raw outputs, so any number can be
traced back to the invocation that produced it.

If external reference values are genuinely required, put them in a clearly separated "external
references" section carrying source URL and retrieval date, and never merge them into the comparison
table or into aggregate scores. When a published number disagrees with the measured one, report the
gap; do not average the two.

## Why

Published leaderboard and model-card scores are snapshots of a different setup: different harness
versions, prompts, sampling parameters, corpus revisions, hardware, and grading rules. They are
stale as soon as either side ships an update, and the reader cannot tell which numbers in a mixed
table are comparable. A pure measured comparison is reproducible - the same commands and corpus can
be re-run - and it keeps the report honest about what was actually observed.

## Exceptions and boundaries

External numbers are valid when the question is explicitly about published claims versus observed
behaviour, or when the requester asks for a reference band. Do not re-search for "fresher" reference
scores after a run has finished; that only trades one stale snapshot for another. Earlier
self-measured results from the same harness and corpus are acceptable if their configuration is
stated; if the harness or corpus changed, re-measure instead of quoting the old run.

## Example

A five-way comparison is assembling its summary table: four rows come from this run's harness and
one row is a vendor figure found by search. The external row used different sampling settings, so
the table silently compares unlike measurements. The fix is to drop that row from the table, keep
the requesters' own deployment as the baseline row, and, because a reference was requested, list the
published figure separately with its source and date. Every remaining cell now maps to a command in
the run log.
