---
{"id":"app-delivery-ops.implementation.allowlist-new-domains-in-no-proxy","title":"Allowlist New Service Domains in the No-Proxy List","stage":"implementation","tech_stack":["agent-ops"],"applies_when":"a host runs with proxy environment variables enabled and the service begins calling a new outbound API domain","severity":"warn"}
---

## When to apply

Apply when a runtime that honors proxy environment variables (fetch-based SDKs and most HTTP clients
do) starts calling a new outbound domain and the host has a system or local proxy configured. Also
apply when a previously working call starts failing after a network change, before suspecting the
remote service.

## Guidance

Every newly integrated domain needs an explicit entry in the no-proxy list on each environment that
will call it - typically a leading-dot suffix so subdomains are covered. Proxy settings are applied
by the HTTP client, not by the network, so a domain absent from the list is tunneled through the
proxy even when it is publicly reachable. When the proxy is a local process, that process becomes a
single point of failure for every unlisted domain.

Edit the environment file on the deployment host, not just your workstation. Back the file up first,
add the entry, and restart the service so the new environment is loaded. Verify from the host itself
with a real request; do not reason about the list from a machine that has no proxy configured.

## Why

Proxy environment variables are all-or-nothing per client: the no-proxy list is the only boundary
between direct and proxied traffic. A routing table that depends on a local proxy process makes
availability of external providers depend on that process being up, converting an unrelated local
crash into an outage of a remote dependency.

## Exceptions and boundaries

Do not disable the proxy wholesale to fix one domain; the allowlist keeps the change scoped and
reviewable. A domain that genuinely must traverse the proxy (for egress policy or auditing) is left
off the list deliberately, with a comment saying so. Test environments need their own entries -
copying a production list can hide a missing local entry.

## Example

A new provider is added to a fallback chain, and its calls fail with connection errors on the server
while the same code works on a laptop with no proxy variables set. The no-proxy list lacks the
provider's domain, so the client routes it through the local proxy - and the proxy is down. Adding
the suffixed domain entry to the host environment and restarting the service restores direct calls.
