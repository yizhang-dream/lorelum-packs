---
{"id":"toolchain-deploy-ops.implementation.pipeline-transcripts-with-provenance","title":"Pipeline Transcripts with Layered Provenance and Idempotent Caches","stage":"implementation","tech_stack":["toolchain","deployment"],"applies_when":"an audio, video, or document pipeline produces derived text - transcripts, translations, rewrites, summaries - from expensive-to-regenerate input that is rebuilt, corrected, or reviewed repeatedly","severity":"warn"}
---

## When to apply

Apply when building or extending a derivation pipeline over recordings or documents whose source
capture cannot be cheaply repeated, and whenever generated claims are later reviewed for accuracy
before delivery.

## Guidance

Model the pipeline as ordered layers with declared provenance, not as one blob of text:

1. Keep the capture layer - verbatim transcript or extracted source text - write-once. Build
   derived paths explicitly from the source path, never by string-replacing suffixes, and never
   write a derived artifact back onto a source-layer path. Guard the raw layer so an accidental
   write fails loudly instead of silently destroying the only copy.
2. Give the capture layer a role no derived layer has: it is the evidence base for every later
   verification. Derived layers can be regenerated from it; it cannot be regenerated from them.
3. Mark reconstructed or generated content by evidence strength. Multi-source agreement may be
   rewritten directly, two-source agreement carries a reconstruction marker, single-source material
   carries an uncertainty marker, and an audit table lists every reconstruction for human review.
4. Cache per item by content hash plus a version key. Key each cached unit by a hash of its exact
   input segment, store the current prompt or schema version alongside it, and bump that version
   whenever the prompt changes - otherwise the cache silently serves outputs from the old prompt.
5. Persist atomically and resume. Write each batch to a temporary file and rename it into place, so
   an interrupted run restarts at the last completed batch instead of at zero. Keep a smoke mode
   (limit N, separate cache namespace) so trial runs never poison the production cache.

## Why

Transcript derivation is expensive and iterative: a long recording costs minutes of accelerator time
plus minutes of model calls, and the same recording is re-run after every prompt or quality fix.
Hash-keyed item caches turn those re-runs into seconds, but only if the key covers everything that
affects the output - input content and prompt version. Provenance is what makes review possible: a
reviewer cannot separate a faithful transcription from a plausible reconstruction unless the layer
and marker say so. And a source layer that a path bug can overwrite is a data-loss incident that
requires re-capturing the audio, not a bug fix.

## Exceptions and boundaries

Short inputs that regenerate in seconds do not need layered caching; the overhead of the discipline
pays off only for expensive capture. Markers are a review aid, not a guarantee - the audit table
must be read, and the pipeline should keep a zero-fabrication rule for content with no supporting
evidence. A cache
key change (prompt version, segment boundaries) legitimately invalidates downstream layers, so
expect a full re-run after such a change and budget for it.

## Example

A build step constructed its output path by replacing a suffix that did not match the input, so a
bilingual draft was written over the verbatim transcript layer; recovery required re-running speech
recognition on the audio. Explicit derived-path construction plus a write-once guard on the capture
layer removes the class. In the same pipeline, sentence-level translation cached by input hash with
atomic batch writes made re-runs of a fully processed lecture take seconds when only a few sentences
had changed.
