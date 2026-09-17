---
{"id":"app-delivery-ops.implementation.layer-prompt-contracts-with-conservation","title":"Layer Prompt Contracts and State Conservation Explicitly","stage":"implementation","tech_stack":["agent-ops","web"],"applies_when":"a model-facing prompt mixes output contract, style guidance, and per-request data in one blob, or the model drops, reshuffles, or silently deletes existing items when transforming structured state","severity":"warn"}
---

## When to apply

Apply when a prompt produces structured output that downstream code parses, when the same instruction is
duplicated across prompt sections, or when transformations are expected to preserve existing state and
sometimes do not. Also apply before shortening a working prompt, so the rules that carry the safety
properties are identified first.

## Guidance

Split responsibilities and keep one authoritative source per rule:

- System layer: output contract (schema, required fields), time anchoring, transformation semantics, and
  quality constraints. This is the only place contract rules live.
- Style layer: field documentation, examples, taxonomy mappings. Never contract rules.
- User turn: data only - the payload plus a single line of time context - with no long-tail instructions.

Put every invariant that matters in the contract, at the highest-salience position, and make it
checkable. Item conservation is the canonical one: every existing item must either appear verbatim in the
output or be named in an explicit removed list; missing and undeclared is a failure. Raise data salience
for inputs the model tends to rewrite: pretty-print the state JSON and annotate item counts. State absolute
requirements that parsers depend on (for example, mandatory phrasing that downstream date inference reads)
and say what to do when a request conflicts with domain rules - apply the safe interpretation and explain.

## Why

Models comply with rules they can see, placed where they are salient; implicit conventions are dropped under
long inputs and reformatting pressure. Without a conservation clause, deletions are indistinguishable from
omissions, and a client-side guard can only repair facts after the fact. Layering prevents the same rule
from existing in three places at three different strengths, which is how prompts drift.

## Exceptions and boundaries

Do not move contract rules into the style layer for brevity, and do not rely on polite phrasing in place of
an explicit declaration requirement. Per-request data never belongs in the system layer. If a client-side
guard exists, treat it as a backstop, not as the primary mechanism - the prompt is what makes the behavior
correct at the source.

## Example

A refinement pass restructures a prompt into the three layers, adds the conservation clause with its
removed list, adds an item-count annotation and indented state JSON, and mandates relative date phrasing.
Overload scenarios that previously lost items reach full compliance while the prompt shrinks by more than
half, with latency and output length both lower than the original.
