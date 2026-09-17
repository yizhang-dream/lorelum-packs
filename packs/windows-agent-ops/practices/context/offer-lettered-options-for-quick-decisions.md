---
{"id":"windows-agent-ops.context.offer-lettered-options-for-quick-decisions","title":"Offer Lettered Options at Decision Points","stage":"context","tech_stack":["agent-ops"],"applies_when":"progress is blocked on a user choice and the moment calls for a question rather than a default assumption","severity":"warn"}
---

## When to apply

Apply at genuine decision points: an ambiguous requirement, a destructive or hard-to-reverse action, a
fork between two designs with different costs. Not for every step - decisions with an obvious default
should be taken and reported, not asked.

## Guidance

- Offer a small set of lettered options (a/b/c), one line each, written as the consequence the chooser
  gets rather than the mechanism you will use.
- Put the recommended option first, but keep all options parallel in tone and length. People read the
  full set; a recommendation steers only when the options are otherwise equal, and a label is not a
  substitute for a stated consequence.
- Batch independent questions into one message so a single short reply resolves them all, and state
  what you will do if no answer arrives.
- Lead with the conclusion, then the reasoning that could change the decision; drop preamble.
- Record the chosen option where the next session will find it, so the decision does not get
  re-litigated.

Keep wording precise about how much was verified: "the mechanism exists" and "the benefit is
demonstrated" are different claims. An option that overstates the evidence makes the choice look
cheaper than it is, and the cost lands later.

## Why

A decision point blocks the whole task, and the cost of the question is dominated by how long the
answer takes to produce, not by how much analysis precedes it. Options with stated consequences can be
answered in one character; open-ended "what do you think" questions push the analysis back onto the
chooser. Batching avoids multiplying that cost by the number of open questions.

## Exceptions and boundaries

Do not ask when the answer is already implied by prior instructions, and do not present options that
are not real alternatives - a fake choice wastes a round trip. If one option is unsafe or
irreversible, say so in its line rather than hiding it in a positional cue. Long deliberation is
sometimes the point: a choice that commits significant resources or is expensive to undo deserves a
full explanation rather than a one-liner.

## Example

A session needs to pick between two storage formats and a migration shortcut. It sends three lettered
lines, recommendation first, each naming the consequence, plus what it will do with no reply. The user
answers with one letter, the session proceeds, and the choice is written into the project notes for
the next session.
