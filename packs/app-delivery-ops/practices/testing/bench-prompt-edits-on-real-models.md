---
{"id":"app-delivery-ops.testing.bench-prompt-edits-on-real-models","title":"Bench Prompt Edits on Real Models with Mechanical Assertions","stage":"testing","tech_stack":["agent-ops"],"applies_when":"a system prompt, rule file, or model-facing instruction is about to be changed - trimmed, restructured, or extended - and the claim that the new version is no worse rests on reading it or on one informal trial","severity":"warn"}
---

## When to apply

Apply to every prompt edit that can change output behavior, including pure shortening. Apply before
declaring a prompt version final, and whenever a prompt-related bug is reported: reproduce it as a
scenario in the benchmark before fixing it.

## Guidance

Treat the prompt as code under test:

- Build an A/B harness that runs the old and new arms against a real model, not a mock, on the same host.
- Cover the failure modes as separate scenarios: generation variants, overloaded input, and each
  transformation path (add, delete, move) as its own case.
- Judge mechanically, not by reading outputs: schema validity, legal time values, no past slots, item
  conservation, required declarations present, required anchor phrasing. Keep latency and output length as
  secondary metrics.
- Require at least three confirmation rounds before freezing. Two rounds at full compliance can collapse
  on the third; treat any difference in outcomes between arms as unresolved.
- Append a scenario whenever a real bug appears, so the matrix grows with the product.

## Why

Prompt behavior is emergent and high variance: a single run, or two lucky runs, cannot distinguish a better
prompt from sampling noise. Mechanical assertions remove reviewer taste from the pass condition, and
scenario coverage prevents "fixed one path, broke another" regressions that a conversational check misses.
A real model is required because the thing being measured is the model's actual compliance.

## Exceptions and boundaries

Do not benchmark with a mocked or stubbed model, and do not let an LLM judge be the pass condition when a
schema or a rule can be asserted directly. Latency numbers are only comparable within one host and model
version; record them as context, not as a gate. The benchmark is a floor, not a substitute for shipping and
observing real usage.

## Example

A prompt rewrite shows two perfect rounds, so it looks done; the third round exposes a conservation failure
that had been hidden by small-sample luck. Adding the conservation clause to the contract, item counts, and
overload resistance, then re-running three rounds with zero differences between arms, produces the version
that ships - with the prompt slightly more than half its original size and lower latency.
