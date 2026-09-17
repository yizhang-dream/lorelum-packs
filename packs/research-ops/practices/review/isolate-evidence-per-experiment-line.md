---
{"id":"research-ops.review.isolate-evidence-per-experiment-line","title":"Isolate Evidence Per Experiment Line","stage":"review","tech_stack":["research"],"applies_when":"assembling or reviewing a comparison, roadmap, or summary that spans a project containing several experimental lines, where evidence from more than one line is available to the writer","severity":"warn"}
---

## When to apply

Apply to any document that evaluates a direction: literature-versus-work comparisons, route
assessments, retrospective summaries, or reviews of such documents. The risk is highest when several
lines produced superficially similar artefacts, because then a sentence can silently borrow support
from the wrong line.

## Guidance

First fix the designated line for the analysis - normally the currently active one, or the line named
by the requester - and make its scope explicit by enumerating which experiments belong to it. Then cite
only that line's own evidence, row by row. Conclusions are not transferable: another line's success
does not validate this line's claim, and another line's failure does not undercut it.

Other lines may appear at most as boundary facts already adopted by the designated line, named as such
and not expanded. If the requester genuinely wants a cross-line synthesis, deliver it as separate,
labelled sections, each with its own line's evidence, and do not merge them into a single verdict or a
single comparison table.

## Why

Mixing lines creates compound claims that no experiment supports: the citation looks like a citation
but the underlying run belongs to a different codebase, dataset, or metric definition. These compound
statements are also impossible to falsify, because no single intervention addresses them. Line
isolation keeps every sentence traceable to a run that actually produced it, which is what makes the
review checkable at all.

## Exceptions and boundaries

Shared infrastructure facts - hardware capacity, cluster availability, common data sources - may be
cited as context in any line, but never as experimental evidence. A line whose results were formally
closed may still be cited as a boundary fact by the active line if that citation already exists in the
line's own record. When the designated line is ambiguous, ask rather than defaulting to a convenient
mix.

## Example

A comparison table against an external paper is drafted by drawing claims from several lines - a
latent-space line, a planner line, and the tracking line. The requester corrects it: the analysis is
only about the tracking line. The table is rebuilt so every row cites a run of that line, with the
other lines removed except for one boundary fact that the tracking line's own report already used.
