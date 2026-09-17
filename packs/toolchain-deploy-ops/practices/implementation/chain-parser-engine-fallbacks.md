---
{"id":"toolchain-deploy-ops.implementation.chain-parser-engine-fallbacks","title":"Chain Parser Fallbacks per Document Instead of Switching the Engine Globally","stage":"implementation","tech_stack":["deployment","toolchain"],"applies_when":"a document-ingest service parses mixed formats or mixed-quality files and one global parser setting is being chosen, or a bulk ingest is unexpectedly slow, leaves the accelerator idle, or silently skips files","severity":"warn"}
---

## When to apply

Apply when designing or operating an ingest service that parses documents from more than one source
(text-layer PDFs, scans, office files), and whenever a bulk ingest is slower than expected, leaves
the accelerator idle, or silently produces no content for some files.

## Guidance

Keep the cheap text-layer extractor as the default route and make every heavier engine a
per-document fallback, never a global switch:

1. Route by declared format support first. If the default extractor reports that it cannot handle a
   format, walk an ordered chain of format-specific converters and take the first one that is
   actually installed and healthy. Do not switch the global engine to solve a per-format problem -
   that removes the fast path for every other file.
2. Route by measured output quality second. After extraction, measure text density (for example
   characters per page) and treat output below a floor as a failed extraction that must trigger the
   heavy OCR path. A file that "succeeds" with near-zero text is the dangerous case, not the one
   that raises an error.
3. Record which engine produced each document's text, so a bad result can be attributed and
   re-parsed deliberately instead of guessed at.
4. Re-read the engine configuration before any bulk run. A one-off switch made to rescue a single
   scanned file tends to persist and quietly reroute the whole corpus.

## Why

A global engine switch turns a per-format workaround into a systemic slow path: heavy OCR pipelines
pay a per-file process cold start and run small batches, so files that extracted in seconds each take
hours while the accelerator sits nearly idle. Silent format skips are worse than slow runs - a loader
that swallows the parser error reports success with empty documents, and the gap is only noticed
later. Per-document decisions keep both properties: the common case stays on the cheap path, and the
expensive path is reached exactly for the files that need it.

## Exceptions and boundaries

For a corpus that is genuinely image-only, route the whole batch through the heavy path
deliberately, approve the cost, and serialize it (see the heavy-rebuild practice). Tune the density
floor per format: figure-heavy documents whose text is only captions can be legitimately sparse, and
a floor that is too high sends real text through OCR. If no member of the fallback chain is
installed and healthy, fail loudly instead of skipping the file.

## Example

A bulk ingest of a few dozen PDFs ran for hours with the accelerator at low utilization. The cause
was a global setting, left over from rescuing one scanned book, that sent every PDF through the OCR
pipeline regardless of text layer. Restoring the default text-layer extractor with the
quality-triggered OCR fallback enabled brought ordinary files back to seconds each, while scanned
files still escalated automatically - without a manual corpus-wide re-parse.
