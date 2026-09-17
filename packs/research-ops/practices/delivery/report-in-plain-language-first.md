---
{"id":"research-ops.delivery.report-in-plain-language-first","title":"Report in Plain Language With Conclusion First","stage":"delivery","tech_stack":["research","ml","agent-ops"],"applies_when":"writing a report, handoff, or status update for a reader who was not in the session, or answering a question in a project dense with internal codenames","severity":"warn"}
---

## When to apply

Apply to status reports, handoffs, review answers, and any document a human reads to decide what happens
next - especially in projects where experiment ids and mechanism abbreviations have accumulated into a
private vocabulary.

## Guidance

Open with the conclusion: one sentence stating what was done or decided, before any context. Explain why,
not only what. Then unpack.

Use zero unexplained internal codenames. If a codename is needed for traceability, put it in parentheses
after a plain-language description, or leave it out; the experiment number is provenance, not the subject
of the sentence. A report that is a chain of bare identifiers carries no information.

Write full sentences with a subject and a verb, one idea each. Split nested clauses into separate
sentences and lift content out of parentheses. Use tables only for short enumerated facts - file lists,
counts, parameter values - never as the explanation itself.

For "what did you accomplish" reports, the effective register is a story for someone who was not present:
describe each mechanism with an analogy, then give the conclusion and the open questions. Keep numbers and
acceptance criteria in the linked document; the report itself carries conclusion, analogy, and decisions
needed.

Treat the vocabulary as a maintenance problem, not a style problem. Codenames multiply because each
document assumes the reader was there yesterday. Keep a glossary that new terms are registered into when
they are introduced, and keep a short plain-language status paragraph that is rewritten whenever the
project changes. Sessions that grow up inside codename-dense documents reproduce the same register in
their own handoffs, so add a short summary section at the top of long-lived documents.

## Why

A reader outside the session cannot recover meaning that was never written down, and the "term
(abbreviation)" annotation alone does not fix it, because the sentence structure itself carries the
project's private grammar. The cost appears as repeated clarification rounds and as decisions taken on a
misread summary.

## Exceptions and boundaries

Internal planning documents may use the project vocabulary freely when the audience shares it; the
obligation applies to anything leaving the session - reports, handoffs, review answers, user-facing
summaries. Precision is not sacrificed: put exact ids, thresholds, and paths in the linked artifact and
keep the narrative claim at the level the reader needs.

## Example

A delivery report that lists completed items as a chain of identifiers and mechanism abbreviations draws
the complaint "please speak like a human". The accepted rewrite tells the same six changes as analogies -
new eyes for the model, a bigger hand for the shop, moving house without breaking the brain - with the
conclusion and open questions at the top and the numbers left in the linked document.
