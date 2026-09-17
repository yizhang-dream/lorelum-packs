---
{"id":"toolchain-deploy-ops.implementation.ingest-papers-via-reference-manager-apis","title":"Ingest Papers Through a Reference Manager's Local API and a Legal Source Ladder","stage":"implementation","tech_stack":["toolchain","deployment"],"applies_when":"papers or reports must be collected, stored with metadata, and handed to a downstream knowledge base or document tool, and the ingest is about to be done by hand or by scraping publisher pages","severity":"warn"}
---

## When to apply

Apply when building or extending an automated paper pipeline: fetch the PDF, register it in a
reference manager with its metadata, then create the per-paper entries in the downstream document
tool. Also apply when a specific paper cannot be fetched from its canonical publisher URL.

## Guidance

Prefer the reference manager's local HTTP API over GUI automation. Recent versions expose a
localhost endpoint that supports both metadata writes and file uploads. The handshake is short: fetch
the server identifier from a plain GET, authorize once through the local authorization endpoint (a
human clicks "always allow" a single time), then send both an API key and the server identifier
header on every write. Exercise the whole chain on one paper before batching, because a
half-authorized client fails in ways that look like a corrupt identifier.

Fetch from legal sources only, and work down a ladder rather than giving up at the first failure: an
open-access copy resolved by DOI, then the publisher's own open archive, then author or institution
pages, then official course pages. If none of those work, substitute a same-topic source and record
the substitution in the pipeline notes so downstream citations stay honest.

Keep per-source quirks in the ladder itself. Direct PDF endpoints of legacy journal archives often
work with a normal browser user agent; conference proceedings sites need the correct file hash
harvested from the listing page; preprint identifiers registered as DOIs may 404 through resolution
services, so keep the direct preprint URL as a fallback. Some metadata services demand a real contact
address and reject placeholder domains, and shared-IP rate limiting needs backoff rather than retries.
Challenge-walled publisher sites, archive domains that fail DNS, and login-gated social networks
should be marked unreachable rather than attacked. Hard red line: shadow libraries and pirate mirrors
are never an option, whatever the deadline.

## Why

Manual collection does not scale and scatters provenance, while GUI automation is brittle against the
reference manager's own interface. The local API is the stable, versioned seam between fetching and
storage. The ladder matters because availability is not uniform: for older literature the canonical
publisher is frequently the least reachable source, and an explicit order stops the pipeline from
"finding" restricted copies under time pressure.

## Exceptions and boundaries

This covers retrieval and storage, not extraction: whether a downloaded file parses depends on scan
quality, and old scans may need a different reader. Respect each publisher's terms and any
institutional agreement; a working URL is not automatically a licensed one. Keep credentials in the
host's standard secret store, never in the pipeline script.

## Example

For a batch of reinforcement-learning papers the pipeline resolves each DOI to an open-access
location, falls back to publisher archives and author pages, and registers the PDFs through the local
reference manager API with metadata. Two papers are unavailable anywhere legal, so same-topic
substitutes are recorded with a note. Papers behind a challenge page or a login-gated network are
logged as unreachable and never fetched through shadow libraries.
