---
{"id":"app-delivery-ops.verification.validate-quotes-as-verbatim-substrings","title":"Validate Quotes as Verbatim Substrings","stage":"verification","tech_stack":["web","ml"],"applies_when":"a product recombines, excerpts, or re-presents user-authored text as quotes, or ships ranking or scoring weights for such content, and the output is about to be shown to users","severity":"warn"}
---

## When to apply

Apply to any feature whose value depends on the quoted text being genuine: excerpting, remixing,
summarizing with quotation, or presenting selected lines from user content. Apply both when building
the emit path and when reviewing a change that touches the selection or assembly logic.

## Guidance

Two rules, both enforced by code:

- Every quoted span must be a verbatim substring of a stored original at emit time. The check runs
  before output leaves the system and blocks on failure. Truncation is allowed; joining fragments
  from different positions or different sources, reordering, or inserting words is forbidden. Keep
  originals immutable and store the quote together with its source identity and character offsets,
  so the check can run again on demand. If the original cannot be loaded, fail closed and do not
  emit.
- Scoring weights must be learned from data, not hand-written. Use weak supervision from observable
  reactions plus online feedback, and validate with a time-split holdout: train on earlier data,
  evaluate on later data, compare against a baseline threshold, and inspect which features carry
  weight before trusting the ranking. A hand-tuned formula may seed a prototype but may not serve as
  the shipped weight set.

Test the checker itself with negative fixtures: a paraphrase, two fragments joined, a reordered
sentence, and an injected word. All must be rejected, and each rejection must be reproducible from
the source store alone.

## Why

Stitched or paraphrased quotes look authentic and are indistinguishable to a reader, so a consumer
cannot detect the failure and the product's core promise - that the text is real - is silently
broken. Enforcement at emit time, rather than during review, makes the guarantee structural. The
same logic drives the second rule: hand-written weights cannot be defended or validated and embed
the author's intuition as if it were evidence; a time split is required because online feedback is
non-stationary and random splits overstate accuracy.

## Exceptions and boundaries

Explicit, documented normalization - whitespace, case, typographic characters - may be applied only
if the checker defines it and it never changes words. Generated summaries, reconstructions, and
translations are not quotes and must be labelled as such rather than passed through this check.
Weights learned from very little data need an explicit floor or abstain path instead of acting
confidently. The check protects authenticity, not correctness of selection: a verbatim quote can
still be a bad choice.

## Example

An assembly step drafts a line by concatenating two fragments that exist in different messages. The
pre-emit check rejects the join because the combined span is not a substring of any single original,
and the pipeline falls back to a single contiguous excerpt from one source. The same pipeline learns
its ranking weights from recorded group reactions, validates them on a time-split holdout against a
baseline, and reports the learned feature weights instead of shipping a hand-written score formula.
