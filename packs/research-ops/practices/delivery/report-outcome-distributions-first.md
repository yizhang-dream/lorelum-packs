---
{"id":"research-ops.delivery.report-outcome-distributions-first","title":"Report Outcome Distributions Before Local Metrics","stage":"delivery","tech_stack":["research","ml","agent-ops"],"applies_when":"a stakeholder asks how strong, how far, or how well a system is doing and you are about to answer with a mean, a single rate, or a per-tier breakdown","severity":"warn"}
---

## When to apply

Apply to any capability or progress question - "how is it doing", "did the change help", "what is the win
rate" - where the answer will be read as a summary of overall performance. Also apply before quoting a
filtered subset of runs as the headline.

## Guidance

Lead with the global distribution over complete runs: quantiles (min, 25, 50, 75, max), a histogram by
depth or score band, the terminal-position buckets showing which stage the run ended in, and the overall
success rate. A single rate is the degenerate right-tail point of that distribution; means and conditional
per-tier numbers come afterwards and must carry an explicit scope label.

Choose the aggregation before looking at results, and name the artifact the numbers came from so they can
be recomputed. When a partial result contradicts the headline, update the headline instead of adding a
qualifier underneath it.

Selection is the common failure mode: a conditional rate over the subset that is easiest to measure, or a
tier that happens to look good, reads as an overall verdict. Keep stratum definitions fixed, report how
many runs each stratum actually contains, and check the realized source distribution - a small sample
drawn with replacement can make a "stratified" evaluation nominal.

## Why

Stakeholders ask capability questions in order to decide what to fund, so the first screen defines what
they believe the system can do. Distributions expose shape - a mass of early failures, a heavy tail, a
bimodal split - while means and single rates hide it. When the distribution is requested under another
name, follow-up rounds get spent before the requester sees anything useful.

## Exceptions and boundaries

Conditional and per-stratum numbers remain valid as diagnostics, and a pre-agreed stratum for a targeted
question is fine, provided the global view is given first or alongside and the scope is labeled. This
practice constrains reporting, not experiment design: if only a mean exists, report it and say explicitly
which distribution is missing.

## Example

Asked how strong a policy is, an agent answers with mean progress plus a conditional success rate on one
difficulty tier, and the requester pushes back twice before receiving the end-of-run picture. The corrected
first screen reports the quantiles, the histogram by depth, the terminal-stage buckets, and the global
success rate, with the per-tier numbers labeled as supplementary.
