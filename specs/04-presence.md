# 04 — Presence

## Overview

Presence describes where peers are and what they are doing within the session. It is informational — the server broadcasts presence updates, but clients are not required to act on them. A client may render cursors, selections, and peer status however it chooses, or not at all.

Presence is ephemeral. It is not persisted and has no effect on document state.

## Messages

### `vemde.presence.update` — Client → Server, Server → All Clients

Sent by a client when its cursor, selection, or status changes. The server broadcasts it to all other peers.

**Protobuf**
```proto
message PresenceUpdate {
  string peer_id  = 1;
  string path     = 2;  // document the peer is currently in, empty if none

  Position cursor    = 3;  // nullable
  Selection selection = 4;  // nullable

  string status = 5;  // "editing" | "viewing" | "idle"
}

message Position {
  uint32 line = 1;
  uint32 col  = 2;
}

message Selection {
  Position from = 1;
  Position to   = 2;
}
```

**JSON**
```json
{
  "peerId": "uuid-v4",
  "path": "docs/intro.md",
  "cursor": { "line": 5, "col": 3 },
  "selection": null,
  "status": "editing"
}
```

`cursor` and `selection` may be `null`. `path` may be empty if the peer has no active document.

## Notes

- Presence updates are best-effort. The server may drop or throttle them under load.
- The server does not need to validate or store presence state. It routes it.
- Clients should not depend on presence for correctness of document state — only for display.
