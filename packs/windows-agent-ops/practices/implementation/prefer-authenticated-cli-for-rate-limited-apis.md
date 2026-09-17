---
{"id":"windows-agent-ops.implementation.prefer-authenticated-cli-for-rate-limited-apis","title":"Prefer the Authenticated CLI Over Anonymous curl for Rate-Limited APIs","stage":"implementation","tech_stack":["windows","git-bash","agent-ops"],"applies_when":"a task needs metadata from a hosted API such as repository counts, release information, or CI status, and the first approach is an anonymous curl or HTTP fetch from a shared or NATed egress IP","severity":"warn"}
---

## When to apply

Apply when an agent fetches read-only metadata from a hosted platform API - star or repository counts,
release lists, package info, pipeline status - and reaches for anonymous `curl` first. Also apply when
a previously working fetch starts returning empty, garbled, or placeholder-looking values with no
obvious error.

## Guidance

Use the platform's authenticated CLI for API access instead of a hand-rolled HTTP call. It carries a
stored credential, handles pagination and redirects, and can filter the response before it reaches
your parser:

```bash
gh api repos/<owner>/<repo> --jq '"\(.full_name) \(.stargazers_count)"'
```

Anonymous requests share a per-IP quota, so on an office NAT, VPN pool, or cloud gateway the budget is
consumed by every other user on that address and small jobs are throttled immediately. The failure is
easy to misread: the throttling message is written to stdout like normal output, so a downstream
parser consumes the error text as data and reports empty fields rather than a failure.

Establish the failure before rewriting anything: check the HTTP status and rate-limit headers once
(`curl -sS -i ...`, or the CLI's own status command) and confirm the credential is present before
blaming the parse step. If no credential exists, authenticate the CLI once instead of pasting a token
into a command line.

## Why

Hand-rolled requests re-implement authentication, pagination, and error detection, and the anonymous
path is exactly the one with the smallest quota. An authenticated CLI moves the quota to an account
bucket, returns a non-zero exit on API errors, and emits structured output, so parsing and failure
detection become the tool's problem rather than yours.

## Exceptions and boundaries

Authenticated access does not mean unlimited: account-level limits still apply, so batch calls and
cache results instead of re-querying per row. Do not paste tokens into shell commands or logs; let the
CLI read its stored credential. Public endpoints that rate-limit per resource rather than per IP may
work anonymously, and endpoints with a documented unauthenticated tier are fine to call directly.

## Example

A script that pulls star counts with anonymous `curl` returns a column of empty values because the
rate-limit notice, not JSON, was on stdout. Switching the same loop to `gh api repos/<owner>/<repo>
--jq ...` returns stable numbers, and the throttling case now surfaces as a non-zero exit instead of
silent garbage.
