# Summary

## What Vemde Is

Vemde is a minimal session protocol for real-time collaborative markdown editing. It defines the message contract between a session server and its connected clients.

## What Vemde Is Not

- A full editor or platform
- A version control system
- A file storage system
- An authentication system
- A transport protocol

## What Vemde Defines

- Session lifecycle (connect, join, leave, disconnect)
- Document sync (open, patch with unified diff, close)
- Presence (cursor position, selection, peer status)
- Contribution registration (commands, status items, decorations)
- Identity (UUID-based, plugin-decorated)
- Plugin message routing (namespaced message bus)
- Input request/response (server asks client for user input)

## What Vemde Leaves to Implementations

- Transport (WebRTC, TCP, HTTP, anything)
- Wire format (Protobuf or JSON, both schemas provided)
- Conflict resolution policy (server implementation choice)
- Connection loss behavior (client implementation choice)
- Plugin loading mechanism (server implementation choice)
- Permissions and access control (server behavior only)
- Async collaboration (version controller's responsibility)
- Authentication (plugin or implementation concern)

## Design Principles

1. **Server is authority.** The server accepts or rejects all operations. Clients reflect server state.
2. **Clients are dumb.** Clients render and send intent. No business logic on the client.
3. **Transport agnostic.** Messages are defined independently of how they travel.
4. **Plugin first.** Everything beyond the session core (including git) is a plugin.
5. **Contributions, not rendering.** The server populates data. The client decides how to display it.
6. **Spec is the source of truth.** All schemas are inline in the spec. No generated artifacts in this repo.
