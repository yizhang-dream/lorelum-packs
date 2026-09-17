---
{"id":"app-delivery-ops.implementation.write-back-every-fallback-level","title":"Write Back Every Level of an AI Fallback Chain","stage":"implementation","tech_stack":["agent-ops"],"applies_when":"you add, remove, or reorder a level in a multi-provider AI fallback chain configured through environment variables, and each level's endpoint, credential, and request flags can be inherited from the level above","severity":"warn"}
---

## When to apply

Apply whenever a service resolves an ordered list of AI providers from environment variables - a
primary level plus one or more fallback levels - and a new level is inserted or the order changes.
Also apply before trusting chain behavior in a deployment you did not configure yourself.

## Guidance

Write every level's endpoint, credential reference, and request flags out in full, even when they
mirror the previous level. Levels that omit a value inherit whatever the level above them has, so a
missing write-back silently chains the new level to the previous level's endpoint or credentials.
This produces failures that look like provider trouble but are really configuration inheritance.

After editing the chain, dump the effective per-level configuration at startup (a log line or a
`--print-config` style command) and diff it against the intended table. Unset one mid-level variable
in a test run and confirm the process fails loudly instead of quietly borrowing its neighbor's value.
Back up the environment file before editing, and keep all levels in one place so the order is
reviewable in a single diff.

## Why

Defaults that inherit from the level above optimize for short configs, but they make the effective
chain a function of edit order rather than an explicit decision. Because each level is then valid in
isolation, misrouting shows up only as a runtime behavior difference - wrong provider answers, auth
failures, or flags meant for one model applied to another - instead of as a config error at startup.

## Exceptions and boundaries

Genuinely shared values (one account intentionally used by two levels) may be inherited if the
startup dump makes the sharing visible. Do not force a new required variable per level for values
that never differ. The goal is eliminating accidental inheritance, not forbidding inheritance as a
feature.

## Example

A chain gains a new top level while existing fallback levels keep working, yet the new level's
requests fail. The startup dump shows the new level has no endpoint of its own and inherited the
previous level's endpoint, which that provider rejects for the new credential. Writing the endpoint
and credential reference back explicitly at every level fixes the routing and makes the next
insertion far less error-prone.
