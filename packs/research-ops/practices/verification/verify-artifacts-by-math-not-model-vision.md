---
{"id":"research-ops.verification.verify-artifacts-by-math-not-model-vision","title":"Verify Artifacts by Math, Not Model Vision","stage":"verification","tech_stack":["simulation","agent-ops"],"applies_when":"a render, recording, or screenshot is about to be accepted as evidence that a simulation or GUI actually ran, and the only reviewer at hand is a model looking at an image","severity":"warn"}
---

## When to apply

Apply whenever visual output - a frame sequence, a screen recording, a GUI screenshot - is
used as evidence that something moved, ran, or produced a result. Also apply when a
multimodal model reports what it "sees" in an image that matters to a verification decision.

## Guidance

Score change with arithmetic before asking any model to look. A changed-pixel fraction over
consecutive frames separates a frozen capture from real motion by orders of magnitude: a
static clip sits near 0.0x% changed pixels while genuine movement runs an order of magnitude
higher. Calibrate that threshold once against a known-dead clip, then reuse it, so the verdict
never depends on a model's opinion.

When a human-style look is genuinely needed, dispatch a vision-capable sub-agent that reads
the local image file and treat its report as one observation, not ground truth. Small blurred
UI text is exactly where vision models hallucinate, so require the check to crop and upscale
the region of interest and to fall back to reading the source code or data when unsure.

Apply the same skepticism to numbers a sub-agent reports about an artifact - frame counts,
motion fractions, step totals. Recompute each one from the artifact before it enters a
report; a single self-reported measurement can be wrong by a large factor while the
surrounding narrative reads perfectly.

## Why

Model vision is a probabilistic description of an image, not a measurement. One-pixel
differences, near-identical frames, and small rendered labels are below its reliable
resolution, so it confidently reports motion that is not present or misses motion that is.
Arithmetic over the artifact has no such failure mode and is reproducible by anyone holding
the same file. Misreading a still frame as "the run is progressing" silently invalidates
every conclusion built on it.

## Exceptions and boundaries

Pixel math answers "did anything change", not "is the change correct" - a moving but wrong
scene scores high. Pair it with a targeted visual or structural check for correctness. Where
an output must match a reference exactly, byte or hash comparison is stronger than either
approach and should be preferred.

## Example

A recording believed to show a long-running simulation scored a changed-pixel fraction of
0.02%, identical to the static-clip baseline: no motion had occurred. The cause lay in input
never reaching the window, not in the renderer, and the arithmetic settled in seconds what two
rounds of model "watching" had misdescribed.
