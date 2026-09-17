---
{"id":"research-ops.delivery.fan-out-doc-trees-with-link-integrity-gates","title":"Fan Out Documentation Trees With Link-Integrity Gates","stage":"delivery","tech_stack":["research","agent-ops"],"applies_when":"a research repository accumulating logs, plans, and status documents has grown past the point where readers and agents can find things by name alone, and new documents are being added without any navigational registry or automated check","severity":"warn"}
---

## When to apply

Apply when a repository's documentation set is large enough that the entry point for a topic is no
longer obvious - typically once several work lines, handoff notes, and status snapshots coexist. Also
apply when an agent or a new collaborator must find the current canonical document for a subject
without being told its path.

## Guidance

Build a fan-out tree with three properties: a root README that carries the navigation rules, the full
tree, and the canonical routes to each topic; a registry that every new document must be attached to
before its content is written; and an automated link-integrity script that checks three things -
registered on the tree, file actually exists, links resolve.

Run the checker before every commit and require it green. New documents must carry a layer-navigation
header and a back-link to their parent domain. Legacy documents are exempt from backfill: they stay
where they are and are not retrofitted, so the gate only binds new material.

If the project already uses a different numbering scheme for content granularity (for example a
handoff document's L-level classification), declare both vocabularies in the root page and state
explicitly that tree depth is not the same axis as content layer. Two unlabelled "layer" numbering
schemes in one repository guarantee that readers and agents will use them interchangeably.

## Why

An unregistered document is invisible: it exists on disk but not in the navigational system, so every
later reader rediscovers it by luck or greps for it again. The integrity check is cheap and mechanical,
which means it can run on every commit, while the failure it prevents - a tree that claims documents
which do not exist or links that lead nowhere - silently destroys trust in the registry. Requiring new
documents to mount first also forces the writer to decide where a document belongs before writing it,
which is when that decision is cheapest.

## Exceptions and boundaries

Append-only run logs, ledgers, and raw status feeds are not tree nodes; mount the documents that
describe them, not every entry. Do not block urgent incident notes on tree registration - publish,
then mount within the same working session. Historical documents without back-links stay exempt; do not
create a cleanup project out of the tree. The gate enforces registration and link integrity, not
document quality or content review.

## Example

A new analysis document is added to a repository that uses this pattern. The author first adds the
node to the root README, then writes the content with a layer header and a back-link to its parent
domain, then runs the checker; it reports the tree entry, file existence, and all links green, and the
commit proceeds. An earlier document that predates the tree is left untouched.
