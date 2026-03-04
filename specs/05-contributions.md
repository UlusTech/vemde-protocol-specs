# 05 — Contributions

## Overview

Contributions are the mechanism by which the server populates client UI. The server (or a plugin running on the server) registers contributions — commands, status items, file decorations — and clients decide how to render them.

Clients are not required to render any particular contribution. A minimal client may render only commands. A terminal client may ignore file decorations. The server sends data; the client decides presentation.

This is modeled after VSCode's contribution points system.

## Contribution Types

### Command

A named action the user can trigger. The client may present it as a button, menu item, keyboard shortcut, or any other affordance.

### Status Item

A piece of text or data to display in a status area. Examples: current branch name, word count, connection status.

### File Decoration

A badge or annotation on a file path. Examples: modified indicator, conflict marker.

## Messages

### `vemde.contributions.register` — Server → Client

Sent when plugins are loaded or when a client first joins a session. Registers one or more contributions.

**Protobuf**
```proto
message ContributionsRegister {
  repeated Command       commands     = 1;
  repeated StatusItem    status_items = 2;
  repeated FileDecoration file_decorations = 3;
  string namespace = 4;  // which plugin is registering these
}

message Command {
  string id    = 1;  // e.g. "git.commit"
  string label = 2;
  string icon  = 3;  // optional, icon identifier string
}

message StatusItem {
  string id    = 1;
  string value = 2;
  string tooltip = 3;  // optional
}

message FileDecoration {
  string path    = 1;
  string badge   = 2;  // short string, e.g. "M" for modified
  string tooltip = 3;
}
```

**JSON**
```json
{
  "namespace": "git",
  "commands": [
    { "id": "git.commit", "label": "Commit", "icon": "git-commit" },
    { "id": "git.branch.create", "label": "New Branch", "icon": "git-branch" }
  ],
  "statusItems": [
    { "id": "git.branch.current", "value": "main", "tooltip": "Current branch" }
  ],
  "fileDecorations": [
    { "path": "README.md", "badge": "M", "tooltip": "Modified" }
  ]
}
```

---

### `vemde.contributions.revoke` — Server → Client

Sent when a plugin unloads or contributions are no longer valid. Clients should remove the listed contributions.

**Protobuf**
```proto
message ContributionsRevoke {
  string namespace         = 1;
  repeated string command_ids      = 2;
  repeated string status_item_ids  = 3;
  repeated string file_decoration_paths = 4;
}
```

**JSON**
```json
{
  "namespace": "git",
  "commandIds": ["git.commit"],
  "statusItemIds": ["git.branch.current"],
  "fileDecorationPaths": ["README.md"]
}
```

---

### `vemde.action` — Client → Server

Sent when the user triggers a contributed command.

**Protobuf**
```proto
message Action {
  string command_id = 1;  // e.g. "git.commit"
  string peer_id    = 2;
}
```

**JSON**
```json
{
  "commandId": "git.commit",
  "peerId": "uuid-v4"
}
```

The server (or the plugin that registered the command) receives this and decides what to do. It may respond with a `vemde.input.request`, a document change, a status item update, or nothing.

---

### `vemde.contributions.update` — Server → Client

Sent when a status item or decoration value changes, without re-registering the full contribution set.

**Protobuf**
```proto
message ContributionsUpdate {
  string namespace = 1;
  repeated StatusItem status_items = 2;
  repeated FileDecoration file_decorations = 3;
}
```

**JSON**
```json
{
  "namespace": "git",
  "statusItems": [
    { "id": "git.branch.current", "value": "feature/intro", "tooltip": "Current branch" }
  ],
  "fileDecorations": []
}
```
