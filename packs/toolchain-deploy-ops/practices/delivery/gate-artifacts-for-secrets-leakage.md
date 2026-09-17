---
{"id":"toolchain-deploy-ops.delivery.gate-artifacts-for-secrets-leakage","title":"Gate Assembled Artifacts for Internal and Secret Leakage","stage":"delivery","tech_stack":["deployment","toolchain"],"applies_when":"a build assembles deliverables from templates, raw source text, and model output, and the pipeline copies, publishes, or pushes the result automatically","severity":"warn"}
---

## When to apply

Apply to any pipeline whose final artifact is assembled and then delivered without a human read:
formatted documents, exports, package contents, pushed payloads. Apply especially when model output
is part of the artifact, because models can re-emit instructions, placeholders, and internal
markers.

## Guidance

Insert a hard gate between assembly and delivery, and make it fail closed:

1. Scan the finished artifact - every output document - not the source. The gate belongs after
   assembly, immediately before the write, copy, or publish step, and a failure must abort all
   three.
2. Define the leak classes explicitly. A practical starter list: unconverted raw source syntax,
   doubled or unescaped markup, broken structure (missing list or table wrappers), empty sections or
   volumes, unresolved placeholders, debug tokens, prompt or instruction fragments, and
   credential-shaped strings if configuration values are ever interpolated into templates.
3. Split error from warning. Intentional protocol or review markers may default to warning so
   internal drafts still build, with a strict mode that promotes warnings to errors for release
   artifacts - the published copy always runs strict.
4. Version the gate, not the content. Keep a version key for the audit rules so extending the rule
   set invalidates gate results without invalidating model-call caches.
5. Test the rules and provide a batch scanner. Unit-test each class against a known-bad sample, and
   have a directory-level scan print a per-artifact verdict table, with an explicit exclusion list
   for artifacts the pipeline did not produce.

## Why

Generated artifacts combine three leak sources: raw markup a converter did not handle, placeholders
from templates that were never filled, and model output that echoes prompt text or internal
annotations. Delivery is automated - a copy to a download folder, a push to a content service - so a
defect that passes assembly ships silently and surfaces either as user-visible garbage or as
internal material exposed to an audience that should never have seen it. A gate after assembly sees
all three sources at once and is the last point at which the artifact can simply be withheld.

## Exceptions and boundaries

Scanning third-party artifacts produces false positives - a title page with no body text reads as an
empty section - so keep non-pipeline artifacts out of the exclusion-free scope or record them as
advisory. Do not weaken a rule because a match is believed to be a false positive: fix the template
or the converter instead, and only then relax the gate. A passing gate says nothing about content
correctness; it only says the defined leak classes are absent, so it complements rather than
replaces a review pass.

## Example

A document builder converted source markup to a display format, but its inline handler missed some
constructs and the assembler emitted them as literal text; with no post-assembly check, the
defective artifact was copied and pushed. Adding a scan over every assembled document that failed
the build on residual raw syntax - with the rule set versioned - caught the leftovers. When a model
upgrade later produced new markup constructs, the same gate caught them before delivery instead of
after.
