# 01 — Overview

## Purpose

Vemde defines the message contract for a live collaborative markdown editing session. A session has one server and one or more clients. The server owns all state. Clients send intent and render what the server describes.

## Message Structure

Every Vemde message has a `type` field that determines its namespace and meaning. The type is a dot-separated string. The first segment is the namespace.

Core protocol messages use the `vemde` namespace: `vemde.session.hello`, `vemde.document.patch`, etc.

Plugin messages use the plugin's own namespace: `git.status`, `myplugin.event`, etc.

### Protobuf Envelope

All messages share a common envelope:

```proto
syntax = "proto3";

package vemde;

message Envelope {
  string type      = 1;  // e.g. "vemde.session.hello"
  string namespace = 2;  // e.g. "vemde" or "git"
  bytes  payload   = 3;  // serialized inner message
}
```

### JSON Envelope

```json
{
  "type": "vemde.session.hello",
  "namespace": "vemde",
  "payload": { }
}
```

The `payload` object is the inner message specific to each `type`. Each spec section defines the payload schema for its messages.

## Message Direction

Messages are either:

- **Server → Client** — the server sends to one or all connected clients
- **Client → Server** — a client sends intent to the server

The protocol does not define request/response pairing at the transport level. The server may respond to a client message with zero, one, or many outbound messages. Correlation, if needed, is handled within specific message pairs (e.g. `vemde.input.request` / `vemde.input.response` share a `requestId`).

## Wire Formats

The protocol provides schemas in both Protobuf and JSON. Implementations choose one. Both are normative — an implementation using JSON must conform to the JSON schema; an implementation using Protobuf must conform to the Protobuf schema.

## Namespaces

| Namespace | Owner | Defined in |
|---|---|---|
| `vemde` | Core protocol | This spec |
| `git` | Example plugin | Plugin implementor |
| (any) | Third-party plugins | Plugin implementor |

The core protocol only routes plugin messages. It does not interpret or validate plugin payloads.
