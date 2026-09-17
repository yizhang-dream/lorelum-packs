---
{"id":"windows-agent-ops.planning.measure-real-adoption-not-stars","title":"Measure Real Adoption, Not Star Counts","stage":"planning","tech_stack":["research"],"applies_when":"a research or selection task asks which products, tools, or frameworks in a category are actually the most used, and the first candidate list is ranked by GitHub stars or another popularity proxy","severity":"warn"}
---

## When to apply

Apply whenever "most used", "largest user base", "real adoption", or "market leader" is part of the
question: landscape scans, build-vs-buy comparisons, competitor studies, and framework selection.
Also apply the moment a shortlist is dominated by open-source repositories, or when the user names a
reference product that does not appear in it.

## Guidance

Treat stars as a developer-attention signal, not a usage metric. Rank candidates by first-hand
adoption indicators instead: MAU/DAU, download or install counts, active deployments, paying seats,
or telemetry published by a neutral tracker. Label vendor-reported figures as self-reported.

Fix the scope before searching: write the exact category and the hard requirements first, then
enumerate candidate forms explicitly - hosted product, desktop app, mobile app, notebook, CLI,
embedded agent, enterprise suite, self-hosted framework. Collect candidates across all of those
forms, filter by the hard requirements, and only then rank the survivors by an adoption indicator.
If the result still reads like a star ranking, or drifts into an adjacent category, recollect rather
than patching the prose.

## Why

A star is a bookmark on a repository; it measures developer interest, not people using something.
Star-sorted pools systematically over-weight open-source, developer-facing frameworks and
under-weight hosted or consumer products, which is precisely the bias that makes a "largest user
base" answer wrong. Adoption numbers live outside repository metadata, so they must be collected
deliberately and cited with their source and date.

## Exceptions and boundaries

In early or niche categories no adoption metric may exist; then state which proxy you used (stars,
package downloads, issue activity) and that it is a proxy. Do not mix vendor-reported and
independently measured numbers inside one ranking without labelling each entry. A star ranking is
legitimate only when the question itself is about open-source community traction.

## Example

A request for the knowledge and learning products with the largest user base returns a page of
self-hosted web frameworks ordered by stars, while the products the user cites as reference points
are hosted apps with published monthly-user figures. Rebuilding the pool with the category fixed and
product forms enumerated surfaces the hosted products, and the ranking switches to an adoption
metric; the star list remains as a separate open-source-traction note.
