---
{"id":"research-ops.planning.stack-research-methodology-skills","title":"Stack Research Methodology Skills by Phase","stage":"planning","tech_stack":["research","agent-ops"],"applies_when":"a project is about to run an experiment-driven research loop and the agent must decide which methodology skills or tooling to install and enable, or is about to design sampling and significance rules ad hoc","severity":"warn"}
---

## When to apply

Apply when standing up a research workflow - a sweep, a comparison study, an ablation campaign - and
when choosing which methodology support to load. Also apply retroactively when an analysis is about
to run without a pre-declared significance rule or without a seed-count estimate.

## Guidance

Select methodology support by research phase and enable only the phases in play now:

- Design: experimental design (factorial or phase-diagram sweeps, multi-seed repetition) plus goal
  definition, so the criteria are frozen before the run.
- Sizing: statistical power before launch, to estimate seeds or sample counts per condition.
- Analysis: statistical testing at judgement time, with the test chosen to match the design.
- Reporting: visualization during analysis, and writing or peer-review support only when the report
  stage starts.

Defer anything that needs extra infrastructure (API keys, notebook toolchains) until its phase
arrives, and prefer the project's own loop over generic tooling. An installed skill fleet is a
capability index, not a plan: installations do not set the thresholds, the workflow does.

## Why

Methodology failures are silent. Too few seeds, a threshold chosen after seeing the data, or a
significance test whose assumptions do not match the design do not crash - they produce a confident
result that later work is built on, after the compute is already spent. Phase-matched skills prevent
that at the cheapest point, while installing everything upfront spends context and maintenance on
phases that may never run.

## Exceptions and boundaries

Scoping pilots that will not support a claim do not need power analysis. Skills are not a substitute
for pre-registration: the skill computes, the process decides what the number means. When a skill's
default test does not fit the design (for example a normal-theory test over a permutation-appropriate
statistic), override it rather than forcing the design to match the tool.

## Example

A phase-diagram sweep is being planned. The workflow enables design, power, analysis, and
visualization support up front; power analysis fixes seeds per cell, the significance rule is written
into the plan before data collection, and visualization runs during the sweep. Literature lookup and
scientific-writing support are left until the report phase, so no phase pays for capability it does
not use.
