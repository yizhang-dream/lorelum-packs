---
{"id":"research-ops.verification.date-and-source-verify-research-reports","title":"Date and Source-Verify Research Reports","stage":"verification","tech_stack":["research"],"applies_when":"an agent is writing, extending, or citing a research report, survey, or literature-backed recommendation, and is about to state a lineage, attribution, or citation that it has not re-checked against a primary source","severity":"warn"}
---

## When to apply

Apply when producing research documents that will be reused across sessions and projects: surveys,
technology comparisons, prior-art sections, direction memos. Apply before repeating any claim from
such a report in a decision discussion, and before adding a citation that came from memory, a blog
post, or another report rather than the source itself.

## Guidance

Name reports `<topic>_<YYYY-MM-DD>.md` under a docs directory, with the date the survey closed.
Different runs on the same topic then coexist, never overwrite each other, and their age is visible
at the call site. Record source and retrieval date for every external claim.

Verify claim by claim against primary sources - the paper, the official documentation, the upstream
repository - and attach the link next to the claim. Where an attribution, identifier, or lineage
cannot be verified, keep it but mark it explicitly as unverified; do not silently upgrade a guess to
a fact, and do not silently delete it either.

Keep an errata section in the report for corrections found after publication. When a decision rests
on a claim from a report, re-read the source instead of quoting the report from memory.

## Why

Research claims decay: affiliations move, identifiers are misremembered, secondary summaries flatten
nuance, and a stale document looks exactly as authoritative as a fresh one. An unmarked unverified
statement is indistinguishable from a checked one, so it propagates into planning and later work as
if it were evidence. Per-claim links and dates make the report auditable and let a reader quarantine
the parts that were never confirmed.

## Exceptions and boundaries

Exploratory brainstorming may speculate freely if the speculation is labelled as such. Widely
settled textbook facts do not need fresh links on every reuse. When a source is paywalled or
offline, cite the entry point (title, venue, year) and mark the claim unverified rather than
dropping it. Honest "unverified" markings are cheaper than a wrong citation and are the expected
output, not a failure.

## Example

A survey compiled from memory asserts a method's lab affiliation and gives a paper identifier for a
related technique. A verification pass against the primary papers finds the affiliation wrong and
the identifier pointing at a different technique. The report is renamed with its closing date, links
are attached per claim, the two errors are corrected with an errata line, and two further claims
that could not be traced are marked unverified instead of being presented as established.
