---
{"id":"toolchain-deploy-ops.delivery.keep-agent-scaffolding-out-of-product-repos","title":"Keep Agent Scaffolding Out of Product Repositories","stage":"delivery","tech_stack":["agent-ops","toolchain"],"applies_when":"a repository that grew inside an agent workflow is being handed off, opened to outsiders, or reviewed, and product content is mixed with the collaboration scaffolding used to build it","severity":"warn"}
---

## When to apply

Apply during hand-off, publication, or any review of a repository that was built by an agent workflow.
Also apply when the tree "reads like a toolchain experiment": newcomers cannot tell the product from the
process, and task cards, dispatch templates, or review checklists outweigh the product's own documents.

## Guidance

Split the tree into two piles before deleting anything. Product: user-facing code, documents an end
user or maintainer needs, infrastructure that does real work (bootstrap scripts, test runners,
environment setup), and verified project facts - engine paths, exact verification commands, invariants,
and known pitfalls with their symptoms. Process scaffolding: task cards, delegation templates, review
checklists, remediation-wave manuals, machine-readable document metadata headers, and prose addressed
to agents.

Delete the second pile, keep the first, and then apply a standing rule: before adding a document to the
product repository, ask whether the product's user or a maintainer of the product needs it. If only the
collaboration workflow needs it, it belongs outside the product - in a methodology repository or in the
workspace that drives the work. Finish by re-running the full verification suite: a cleanup that breaks
the safety net is a regression, not a cleanup.

## Why

Scaffolding accumulates because it is useful while building, not because it is a deliverable, and it
distorts how the repository reads. Future maintainers cannot separate product requirements from process
artifacts, and the process artifacts go stale immediately because they encode one way of working at one
point in time. Facts survive that turnover; methodology does not.

## Exceptions and boundaries

Do not strip facts along with the process - engine paths, verification commands, invariants, and pitfall
lists are load-bearing product knowledge, and losing them makes the repository unusable for the next
person. Tools that do work (bootstrap, selective test runners) stay. User-level or workspace-level agent
configuration is outside the product repository's scope and is not part of this cleanup.

## Example

A game repository had accumulated six start-up task cards, a metadata header on every document, and
delegation wording leaked into code comments. The owner's ruling was that it is a teaching product, not
a methodology testbed. The cleanup deleted the cards and headers and cut the agent instruction file by
more than half, while keeping the engine paths, the smoke-test commands, the naming and unit invariants,
and the pitfall notes. All suites were re-run green afterwards, and the repository's entry point now
describes the product rather than the workflow that produced it.
