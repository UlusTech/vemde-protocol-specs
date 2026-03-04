# Architecture

## Core Concept

Vemde is a session protocol. A session is a live, ephemeral context in which one or more peers collaboratively edit markdown documents. The session has a single server, which is the absolute authority for all state. Clients are dumb — they render what the server describes and send user intent back.

This is intentionally modeled after VSCode's Live Share extension and the Language Server Protocol.

```
┌─────────────────────────────────────────┐
│              Vemde Server               │
│  owns session state, documents, plugins │
└────────────────┬────────────────────────┘
                 │  Vemde Protocol
       ┌─────────┼─────────┐
       ↓         ↓         ↓
   [Client A] [Client B] [Client C]
   (host)     (guest)    (guest)
```

## Layering

Vemde (`vemde.*`) is the core protocol. It handles only:

- Session lifecycle
- Document state and diffs
- Presence (cursors, selections)
- Contributions (plugin-registered commands and UI data)
- Identity
- Plugin message routing

Everything else — git, markdown rendering, permissions, authentication — is a plugin or implementation concern. Plugins use their own namespace over the same message bus.

```
┌────────────────────────────┐
│         vemde.*            │  ← core protocol
└────────────────────────────┘
     ↑ plugins register via vemde.contributions
┌──────────┐  ┌──────────┐  ┌──────────────┐
│  git.*   │  │  myplugin.*│  │  anything.* │
│ (plugin) │  │  (plugin) │  │  (plugin)   │
└──────────┘  └──────────┘  └──────────────┘
```

## Server Authority

The server is the single source of truth. Clients send intent; the server accepts or rejects it and broadcasts the resulting state. Clients must reflect server state exactly. A rejected operation must be rolled back by the client.

What the server chooses to accept, reject, or broadcast is an implementation decision — the protocol does not mandate conflict resolution policy, permission models, or session rules.

## Transport Independence

The protocol defines messages and their schemas. It says nothing about transport. Implementations may use WebRTC, TCP, HTTP, Unix sockets, or any other mechanism. The message format (Protobuf or JSON) is also an implementation choice — both schemas are provided in the spec.

## Async Work

Vemde is a live session protocol only. Asynchronous collaboration — working offline, syncing later, merging diverged work — is outside Vemde's scope. Implementations that use a version control system (such as git) handle async work through that system directly.

## Sessions

A session is always owned by a server. The server may be:

- A dedicated always-on process (a team's shared Vemde server)
- An embedded server in a desktop client (a user sharing their machine)
- Any other process that implements the protocol

If the server becomes unavailable, the session is interrupted. Resumption behavior is a client implementation choice.
