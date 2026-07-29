---
name: Inspect Cilium endpoints and identities
description: >-
  Use the cilium-agent REST API to list managed workload endpoints, inspect an
  individual endpoint's status and policy, and resolve the security identities and
  labels that govern them.
api: openapi/isovalent-cilium-agent-openapi.yml
transport: unix-socket
operations:
- GET /endpoint
- GET /endpoint/{id}
- GET /endpoint/{id}/config
- GET /endpoint/{id}/labels
- GET /identity
- GET /identity/{id}
---

# Inspect Cilium endpoints and identities

The cilium-agent API is served over a local Unix domain socket
(`unix:///var/run/cilium/cilium.sock`, base path `/v1`), reachable from within the
agent's node. All requests and responses are `application/json`. There is no
API key or OAuth; access is controlled by socket locality (see
`authentication/isovalent-authentication.yml`).

## Steps

1. **List endpoints.** `GET /endpoint` returns all endpoints with metadata. Each
   endpoint carries its `id`, networking, `status.identity`, `status.labels`, and
   `status.policy`.
2. **Inspect one endpoint.** `GET /endpoint/{id}` for a specific endpoint. Use
   `GET /endpoint/{id}/config` for its mutable datapath configuration and
   `GET /endpoint/{id}/labels` for its label set.
3. **Resolve the identity.** Take `status.identity` from the endpoint and call
   `GET /identity/{id}` to see the labels that define that security identity.
   `GET /identity` lists all identities.
4. **Handle errors.** Expect the `{code, message}` `Error` envelope. `404` means
   the endpoint/identity does not exist; `429` signals the agent is at capacity;
   `500` is an agent-side failure. See `errors/isovalent-problem-types.yml`.

## Notes
- This is a read-oriented flow. Mutations (`PUT/PATCH/DELETE /endpoint/{id}`) are
  not idempotent-key protected — see `conventions/isovalent-conventions.yml`.
