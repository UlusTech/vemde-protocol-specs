# 07 — Plugins

## Overview

Plugins extend the server's behavior. They run server-side and communicate with clients using their own message namespace over the Vemde message bus. Vemde routes plugin messages — it does not interpret or validate their payloads.

A client that does not know about a plugin may ignore its messages. A client that does know about a plugin may render its contributions, respond to its input requests, and handle its custom messages.

## Plugin Namespaces

Each plugin owns a namespace — a short dot-separated prefix for all its messages. Examples:

| Plugin | Namespace |
|---|---|
| Git integration | `git` |
| Word count | `wordcount` |
| Spell check | `spellcheck` |
| Custom org plugin | `myorg.review` |

Namespaces must not start with `vemde`. That prefix is reserved for the core protocol.

## Plugin Message Routing

Plugin messages travel inside a `vemde.plugin.message` envelope.

### `vemde.plugin.message` — Server ↔ Client (both directions)

**Protobuf**
```proto
message PluginMessage {
  string namespace = 1;  // e.g. "git"
  string type      = 2;  // e.g. "git.status"
  bytes  payload   = 3;  // plugin-defined, serialized however the plugin wants
}
```

**JSON**
```json
{
  "namespace": "git",
  "type": "git.status",
  "payload": {
    "branch": "main",
    "modified": ["README.md"],
    "staged": []
  }
}
```

The payload schema is entirely defined by the plugin. Vemde makes no assumptions about it.

## Input Request / Response

Plugins that need user input use the core `vemde.input.*` messages. This avoids plugins needing to define their own input mechanisms.

### `vemde.input.request` — Server → Client

Sent when a plugin needs the user to provide input before continuing. The client collects the input using whatever UI it chooses (modal, inline prompt, command palette, etc.) and responds with `vemde.input.response`.

**Protobuf**
```proto
message InputRequest {
  string request_id = 1;  // correlation ID
  string context    = 2;  // hint for the client, e.g. "git.commit"
  repeated InputField fields = 3;
}

message InputField {
  string id       = 1;
  string type     = 2;  // "text" | "bool" | "select"
  string label    = 3;
  bool   required = 4;
  repeated string options = 5;  // for "select" type
}
```

**JSON**
```json
{
  "requestId": "r1",
  "context": "git.commit",
  "fields": [
    { "id": "message", "type": "text", "label": "Commit message", "required": true },
    { "id": "amend",   "type": "bool", "label": "Amend last commit", "required": false }
  ]
}
```

---

### `vemde.input.response` — Client → Server

**Protobuf**
```proto
message InputResponse {
  string request_id          = 1;
  map<string, string> values = 2;
}
```

**JSON**
```json
{
  "requestId": "r1",
  "values": {
    "message": "Fix heading typo",
    "amend": "false"
  }
}
```

---

### `vemde.input.cancel` — Client → Server

Sent if the user dismisses the input request without completing it.

**Protobuf**
```proto
message InputCancel {
  string request_id = 1;
}
```

**JSON**
```json
{
  "requestId": "r1"
}
```

## Plugin Loading

How plugins are loaded is a server implementation decision. The protocol only defines how plugins communicate once loaded. The official Vemde server uses dynamic import of local modules.

## Plugin Discovery

When a client joins a session, the server sends `vemde.contributions.register` messages for all loaded plugins. This tells the client what namespaces and commands are active. The client may use this to decide which plugin messages it knows how to handle.
