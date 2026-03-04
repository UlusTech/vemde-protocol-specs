# 03 — Document

## Overview

A document is a markdown file managed within a session. The server owns all document state. Clients propose changes via patches; the server accepts or rejects them and broadcasts the result.

Documents are identified by a path (relative to the repo root). Each document in a session has a `baseRev` — a content hash of its current state. Patches must include the `baseRev` they were computed against. The server rejects patches with a mismatched `baseRev`.

## Patch Format

Patches are unified diffs. Example:

```diff
@@ -3,4 +3,5 @@
 ## Intro
-This is old text.
+This is new text.
+Added a line.
 ## Next
```

## Lifecycle

```
Server                          Client
  |                               |
  |-- vemde.document.opened ----->|  server opens a doc, sends full content
  |                               |
  |<-- vemde.document.patch ------|  client proposes a change
  |-- vemde.document.patch ------>|  server accepts, broadcasts to all peers
  |   (or)                        |
  |-- vemde.document.rejected --->|  server rejects, client must roll back
  |                               |
  |-- vemde.document.closed ----->|  server closes the document
```

## Messages

### `vemde.document.opened` — Server → Client

Sent when the server opens a document for the session. Contains full current content.

**Protobuf**
```proto
message DocumentOpened {
  string path     = 1;  // relative path, e.g. "docs/intro.md"
  string content  = 2;  // full markdown content
  string base_rev = 3;  // content hash of current state
}
```

**JSON**
```json
{
  "path": "docs/intro.md",
  "content": "# Introduction\n\nHello.",
  "baseRev": "a1b2c3"
}
```

---

### `vemde.document.patch` — Client → Server, Server → All Clients

Used in both directions. Client sends a proposed patch; server broadcasts the accepted patch to all peers.

**Protobuf**
```proto
message DocumentPatch {
  string path     = 1;
  string base_rev = 2;  // the rev this patch was computed against
  string diff     = 3;  // unified diff string
  string peer_id  = 4;  // set by server on broadcast, empty on client send
}
```

**JSON**
```json
{
  "path": "docs/intro.md",
  "baseRev": "a1b2c3",
  "diff": "@@ -1,1 +1,1 @@\n-# Introduction\n+# Intro",
  "peerId": ""
}
```

On broadcast, the server sets `peerId` to the originating peer's ID and updates `baseRev` to the new content hash after applying the diff.

---

### `vemde.document.rejected` — Server → Client

Sent to the client whose patch was rejected. The client must roll back the change.

**Protobuf**
```proto
message DocumentRejected {
  string path     = 1;
  string base_rev = 2;  // the baseRev the client should revert to
  string reason   = 3;  // human-readable, optional
}
```

**JSON**
```json
{
  "path": "docs/intro.md",
  "baseRev": "a1b2c3",
  "reason": "baseRev mismatch"
}
```

---

### `vemde.document.closed` — Server → All Clients

Sent when the server closes a document. Clients should remove it from the active session.

**Protobuf**
```proto
message DocumentClosed {
  string path = 1;
}
```

**JSON**
```json
{
  "path": "docs/intro.md"
}
```

## Conflict Resolution

Conflict resolution policy is a server implementation decision. The protocol provides only the `vemde.document.rejected` message for the rejection case. The official Vemde server uses first-write-wins: the first patch received for a given `baseRev` is accepted; subsequent patches for the same `baseRev` are rejected.
