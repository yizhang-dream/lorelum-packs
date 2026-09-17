---
{"id":"research-ops.delivery.gate-external-report-phases","title":"Gate External Reports to Approved Project Phases","stage":"delivery","tech_stack":["research"],"applies_when":"an external report, summary, or application material is being written about a project that has several phases with different publication status, and facts from internal or unpublished phases are available to the writer","severity":"warn"}
---

## When to apply

Apply before writing any document that leaves the project: progress reports, training or course
reports, applications, shared summaries. It matters most when the project's later-phase work is
technically the more interesting material, because that is exactly when an unfiltered draft leaks
unpublished results.

## Guidance

Maintain an explicit phase boundary and filter content by it before drafting, not after. Decide which
phases may be described externally and which must not appear at all, together with any standing
lexical bans (unpublished paper titles, venue names, competitor comparisons). Keep a matching
fact-level policy for ambiguous claims: hardware that was only exercised in simulation, features that
were demonstrated versus merely attempted, deliverables that count versus those that do not.

Where the boundary or a fact inside it is uncertain, resolve it by asking, one question at a time,
rather than guessing a safe-sounding version. A single-question confirmation loop keeps the writer
from inventing scope; batching several uncertainties into one broad question tends to produce vague
answers and a draft that has to be redone.

## Why

Phases are separated for real reasons: unpublished work can be scooped, a report that claims more than
was verified misrepresents the project's capability, and later-phase material can be mistaken for
results of the phase being reported. Mixing phases once costs a rewrite and, worse, can burn
publication priority or credibility. Filtering at draft time is cheap and reversible; correcting an
already-submitted report is neither.

## Exceptions and boundaries

Internal documents may cross phases when they are clearly labelled as internal and the crossing is
stated. A project decision owner can move the boundary; then update the rule rather than relitigating
each document. Do not over-filter: verified facts that fall inside the allowed phase should be stated
plainly instead of hedged into vagueness. Once an uncertainty is confirmed, fold the answer into the
policy so the next document does not need to ask again.

## Example

A report draft includes results from an internal self-training phase alongside the deployment
reproduction phase. The reviewer maps both phases, keeps only the deployment phase in the external
report, drops the unpublished-method references, and asks one question about whether a hardware
component was really exercised on the physical robot before asserting it. The corrected draft states
only the verified in-simulation scope.
