---
{"id":"research-ops.implementation.profile-before-optimizing-hot-paths","title":"Profile Before Optimizing Hot Paths","stage":"implementation","tech_stack":["ml","research"],"applies_when":"a training or simulation loop is judged slow and a language rewrite, GPU batching, or a native extension is proposed as the fix","severity":"warn"}
---

## When to apply

Apply when someone proposes a rewrite (another language, full simulator rewrite), GPU batching, or a
native extension to fix slowness, and when a microbenchmark reports a large speedup that has not been
confirmed end to end.

## Guidance

Measure the shape of the cost before choosing a lever:

- Profile at whole-run (wall) level and attribute wall time by phase: decision/search, real simulation
  logic, environment transitions, model inference. The dominant share decides the lever.
- Add a decision-level profile that counts interpreter-mechanism calls (copy, attribute, dict lookups)
  to separate framework overhead from business logic.
- Compare configurations as paired arms on identical seeds so any difference is attributable.
- Then climb a ladder, cheapest rung first: configuration and data-structure specialization, reusing
  structures across branches, narrow routing that skips expensive work, extracting a single hot kernel
  into a native extension, and only then a full rewrite.
- Re-profile after every rung: the remaining share of the previously dominant component tells you
  whether the next rung is still worth it.
- Confirm microbenchmark wins at wall level. One isolated fix measured about 0.39x for the optimized
  component, but at wall level a side cost of the same fix made the full path slower, which reordered
  the priorities; only the run-level measurement exposed it.
- Price the rewrite honestly: it invalidates the identity chain of every artifact defined against the
  current implementation, adding weeks or months of re-validation. When the remaining unoptimized share
  is small, adding compute is often an order of magnitude cheaper.

## Why

Guessed hot spots are wrong at this scale. In a measured decision loop the actual simulation logic was
low single-digit percent of wall time and model inference under ten percent, while object copying
dominated; a rewrite of the simulator could therefore capture only a few percent, and GPU batching was
not a lever at all. Profiling turns "the language is slow" into a ranked list of levers with known
payoff.

## Exceptions and boundaries

Keep profiling when a phase is large enough that a rewrite could plausibly matter - for example when
search becomes the wall. If wall-level and micro-level pictures disagree, treat the contradiction as a
finding to explain, not as noise. Re-profile whenever the workload changes shape; a copy fix can move
the dominant cost elsewhere, raising or lowering the payoff of the next rung.

## Example

An experiment loop attributed 94-99% of wall time to decision-time search. A dedicated copier replaced
generic deep copy and cut wall time roughly in half; after that, copying held only a quarter to a third
of wall time. That measurement demoted a native-kernel extraction to "only if needed" and kept a full
rewrite off the roadmap - both had been proposed on per-call cost intuition alone.
