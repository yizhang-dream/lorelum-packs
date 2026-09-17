---
{"id":"research-ops.delivery.keep-docs-function-first","title":"Keep Docs Function-First","stage":"delivery","tech_stack":["agent-ops","research"],"applies_when":"writing or updating long-lived documents such as research notes, module docs, and indexes that people and agents will reopen months later","severity":"warn"}
---

## When to apply

Apply when creating or updating long-lived documents: design and research notes, module documentation,
index files, preregistrations. Also apply when answering any "what is this?" question about an existing
artifact, in chat or in a report.

## Guidance

Put function before taxonomy:

- Open every long-lived document with a plain-language block in the first screen: what this is, why it
  exists, and what it is used for. Status banners and terminology come after it.
- In index files, make the one-line summary column start with the function or purpose, not the topic
  name, so a reader scanning the index learns what each artifact does and not only what it is about.
- Answer questions about an artifact with one sentence of functional positioning before any detail,
  status, or metrics.
- For frozen or criteria documents, label the intro block as not part of the frozen content and insert
  it only: never edit frozen sections, and record the insertion in a change-log line.
- A good file name is not a substitute for an in-file functional snapshot. An agent should not have to
  combine a file name with git-history archaeology to reconstruct what an artifact is.

## Why

Append-only histories and status-heavy documents make it hard to tell which document is authoritative
and what each one is for. The reader's first question is functional - what is this and why does it exist
- and answering it in the first screen saves that cost on every later visit. Without the block, readers
re-derive context from names, logs, and diffs, and the same cost is paid again every session.

## Exceptions and boundaries

Append-only logs, ledgers, and machine-generated artifacts do not need the block; short-lived scratch
notes do not either. The intro block is an insertion only - do not rewrite frozen text to make room for
it - and documents that define criteria must state that the block is not part of those criteria.

## Example

An agent reconstructed what a research component was from its file name plus git history and reported
that reconstruction, and the owner rejected it: the name was fine but the purpose was nowhere near the
top of the file. The fix became a published convention - long-lived docs carry a what/why block in the
first screen, index one-liners start with function - and the first application inserted a block at the
top of a frozen preregistration, left the criteria sections verbatim, and passed independent review.
