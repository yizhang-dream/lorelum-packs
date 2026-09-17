---
{"id":"toolchain-deploy-ops.delivery.dual-license-code-versus-content","title":"License Code and Authored Content Separately","stage":"delivery","tech_stack":["web","deployment"],"applies_when":"a repository publishes both source code and substantial authored content, or content derived from a third-party work, and the project is being opened up, redistributed, or handed to a wider audience","severity":"warn"}
---

## When to apply

Apply when the distribution context changes - code is published on a platform, the repository goes
public, or the deliverable is handed to an outside audience. Apply before announcing the repository as
open source, and when adding a third-party body of material to a repository that already carries a
license.

## Guidance

Split the grant by artifact type. Use a permissive software license for code - markup, scripts, styles,
build tooling - and a content license such as CC BY for authored content such as explanatory text,
question banks, derivations and diagrams, each in its own file, and state in each file exactly what it
covers.

Add a layered notice file with three parts: which artifacts fall under which license, which material is
third party and explicitly outside the grant with its rights holders named, and practical reuse
guidance. Reference the notice from the README and from a visible place in the product itself, and
document the split in the contributing guide so a contributor knows which terms their change falls
under.

## Why

One license for a mixed repository either over-grants or under-grants. Over-granting claims rights over
material the project does not own - formulas, structure, videos belonging to an upstream author - which
is the exposure that actually matters. Under-granting leaves the code's reuse terms ambiguous, so
well-behaved downstream users cannot tell whether they may build on it. Naming the excluded third-party
material explicitly is what separates a compliant repository from a takedown risk, and it costs one
clearly written page.

## Exceptions and boundaries

A repository with no authored content does not need a content license, and one with no third-party
material does not need an exclusion section. When the upstream material is itself permissively
licensed, an attribution line in the notice is usually enough - do not invent restrictions the upstream
license does not impose. License files are delivery artifacts: they must ship in the same release as
the content they describe, not in a later cleanup pass.

## Example

A teaching site ships its own code, its own explanatory content, and structure and formulas derived
from a published book plus an official video course. The release adds a permissive license for the
code and a CC BY license for the authored content, and a notice naming the upstream book and video
course as excluded third-party material. The README and a footer link point at that notice, and the
contributing guide states the split, so a contributor adding a lesson knows which terms govern it.
