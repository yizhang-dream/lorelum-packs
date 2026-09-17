---
{"id":"narrative-writing.verification.recompute-ratios-without-trailing-metadata","title":"Recompute Ratios Without Trailing Metadata","stage":"verification","tech_stack":["writing"],"applies_when":"a per-act ratio or balance check fails on the final act of a script whose file ends with a version footer, revision list, or metadata block","severity":"warn"}
---

## When to apply

Apply when a per-act narration-versus-dialogue balance check reports the last act out of band and the
script file ends with a version footer, revision list, or similar trailing block. Also apply before
rewriting the last act to "fix" such a failure.

## Guidance

- Subtract the trailing metadata block from that act's narration count and recompute before drawing
  any conclusion. Check the footer explicitly: list lines such as version entries under a version
  heading parse as prose to a line-based counter.
- Compare corrected numbers against the threshold, then decide whether a real fix is needed. In one
  measured case a final act's narration total of hundreds of characters was mostly footer, and
  another act's entire narration count was footer with zero real narration lines; both passed after
  subtraction.
- Never delete or rewrite body prose to satisfy a raw count.
- When the tool is eventually fixed, do it as a proper change: add a skip rule for the footer block,
  regress against a golden sample, keep previously reported results stable, and record the behavior
  change. Until that lands, treat the failure as an artifact.
- Track the fix as tool debt with its preconditions rather than blocking the current writing pass.

## Why

A checker that counts narration characters per act cannot distinguish prose from file metadata. Any
script carrying a footer therefore fails its last act by construction, and the failure recurs on
every pass. Unrecognized, the artifact invites the natural next step - cutting narration to hit the
ratio - which removes real writing to satisfy a counting bug.

## Exceptions and boundaries

Subtract trailing metadata only. Scene-level notes or inline comments that the audience-facing script
legitimately contains are not metadata and can genuinely skew a ratio, so they deserve a real fix.
The check is also not a blanket excuse: recompute first, and if the corrected ratio still fails, the
act has a real problem that the normal revision path should handle.

## Example

A per-act check flagged the final act of several scripts in the same pass. In one, the act's 570
narration characters were 346 footer; in another, the act's 150 "narration" characters were entirely
footer and the body had no narration lines at all. Subtracting the footer brought both to about the
expected ratio. The team logged a tool change - skip rule, golden-sample regression, no flipping of
recorded results - and moved the correction off the critical path.
