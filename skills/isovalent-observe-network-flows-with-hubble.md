---
name: Observe network flows with Hubble
description: >-
  Use the Hubble Observer gRPC API to stream and query real-time network flows,
  agent events, and cluster node/namespace observability from Cilium's eBPF
  dataplane.
api: grpc/isovalent-hubble-observer.proto
transport: grpc
operations:
- Observer.GetFlows
- Observer.GetNodes
- Observer.GetNamespaces
- Observer.ServerStatus
---

# Observe network flows with Hubble

The Hubble Observer service is a gRPC API (see
`grpc/isovalent-hubble-observer.proto`) typically reached through Hubble Relay
(`hubble-relay`, default port 4245 via `hubble` CLI port-forward). Relay-to-peer
communication supports optional mTLS (`authentication/isovalent-authentication.yml`).

## Steps

1. **Check the observer is up.** Call `ServerStatus` (unary) to confirm the
   Hubble server is serving and to read flow-buffer and peer counts.
2. **Discover scope.** `GetNodes` lists the nodes reporting flows;
   `GetNamespaces` lists namespaces currently observed — use these to scope a
   query.
3. **Stream flows.** Call `GetFlows` with a `GetFlowsRequest` (filters on
   namespace, labels, verdict, protocol, etc.). It returns a **stream** of
   `GetFlowsResponse` messages — consume the stream and stop when `number` is
   reached or the follow flag is cleared.
4. **Correlate to identities.** Flow records carry source/destination identity and
   label metadata; cross-reference with the cilium-agent `GET /identity/{id}`
   endpoint (see `skills/isovalent-inspect-endpoints-and-identities.md`).

## Notes
- `GetFlows`, `GetAgentEvents`, and `GetDebugEvents` are server-streaming RPCs;
  design for backpressure and long-lived connections.
