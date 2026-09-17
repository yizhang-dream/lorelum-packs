---
{"id":"toolchain-deploy-ops.verification.accept-rendering-at-node-level-baselines","title":"Accept Heuristic Rendering Changes at Node Level Against a Real-Corpus Baseline","stage":"verification","tech_stack":["toolchain","deployment"],"applies_when":"a heuristic that rewrites free text into math or other markup is being changed, and acceptance is about to be judged from regex or span-level counts over a few hand-written samples","severity":"warn"}
---

## When to apply

Apply whenever a text-to-markup heuristic (auto-wrapping math delimiters, escaping characters,
rewriting link targets) is modified, and whenever the change must be proven not to regress a
renderer that fails loudly on invalid input. Also apply before accepting any change whose only
evidence is a handful of illustrative samples.

## Guidance

Accept at node level, not at span level. Parse the text with the real parser, walk every math or
inline-math node it produces, and feed each node value to the real renderer with its strict error
mode on. Count nodes that throw - never the number of delimiters or regex matches, which stay blind
to nested delimiters and to markup absorbed into surrounding list or quote syntax.

Compare against a baseline harvested from the real corpus, not against zero. Extract the message
history production actually generated, run the new code over it, and require the current failure set
to be a strict subset of the baseline set, with zero structural breaks and zero orphan symbols.
Report the comparison as a set relation; "no new failures" is a much weaker and much more misleading
statement than "current failures are a subset of what already existed".

Gate on legality, not on best effort: if the candidate transformation does not render cleanly, leave
the source untouched rather than wrapping it into a visible error. Keep the detection list and the
transformation list separate, because a construct can be worth detecting while the renderer cannot
render its wrapped form at all.

Measure the heuristic's false-positive surface on the same corpus. Symbols that appear inside
ordinary prose (a separator dot inside a name) must never be seeds for wrapping, and repaired
delimiters must be checked for double-wrapping before the fix is accepted.

## Why

Span-level metrics reward exactly the wrong thing: they rise when delimiters are added, including
additions that break nested markup a regex cannot see. Only the parser knows node boundaries, and
only the renderer knows whether the result is legal. A real-corpus baseline separates "this change
introduced a failure" from "this corpus was already imperfect", which is the question the acceptance
decision actually depends on.

## Exceptions and boundaries

Corpus size limits confidence: a subset relation over a few hundred messages is evidence, not proof,
so extract the largest sample the store allows and state the sample size. Do not normalize old
baseline failures away - listing them as known debt with their identifiers keeps later changes
accountable to the same comparison. Node-level checks do not cover layout, spacing or visual
quality; those still need a rendered sample review.

## Example

A heuristic that auto-wraps bare math in assistant messages is applied to a 264-occurrence corpus.
Span-level counts look excellent, yet two review rounds reject the change: node-level rendering finds
a dangling subscript from an escaped underscore, list markers absorbed into formulas, and mid-message
delimiters that lost their protection. Re-running with the parser and the strict renderer over 151
real messages yields 8 pre-existing failures; after the fix the current set is 4 - a strict subset -
with zero structure breaks and zero orphan symbols. That comparison, not the span counts, is the
acceptance record.
