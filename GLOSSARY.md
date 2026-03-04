# Glossary

**Session**
An ephemeral, live collaborative context hosted by a Vemde server. A session has one or more peers and zero or more open documents. Sessions are not persisted by the protocol — persistence is an implementation concern.

**Server**
The process that owns a session. The server is the absolute authority for all session state. It accepts or rejects client actions, applies document patches, and broadcasts state to all peers.

**Client**
A process that connects to a session. Clients render server-described state and send user intent to the server. Clients have no authority — they reflect what the server says.

**Peer**
A connected client within a session. Each peer has a UUID and optional plugin-contributed identity metadata.

**Document**
A markdown file managed within a session. Documents have a path, content, and a current revision (`baseRev`).

**baseRev**
A content hash of a document's current state in the session. Used to verify that a patch applies to the correct version. The server rejects patches whose `baseRev` does not match.

**Patch**
A unified diff applied to a document. Sent by a client to propose a change, or by the server to distribute an accepted change.

**Contribution**
A piece of UI data or a command registered by a plugin. The server sends contributions to clients; clients decide how to render them. A client may ignore contributions it does not understand.

**Command**
A named action registered as a contribution. Clients may present commands as buttons, menu items, keyboard shortcuts, or other affordances. When triggered, the client sends a `vemde.action` message to the server.

**Plugin**
A server-side module that extends session behavior. Plugins communicate with clients using their own message namespace over the Vemde message bus.

**Namespace**
A dot-separated prefix that scopes messages to a plugin or the core protocol. Example: `vemde.*` is the core namespace. `git.*` is a hypothetical git plugin namespace.

**Transport**
The mechanism used to carry Vemde messages between server and client. The protocol is transport-agnostic.

**Wire Format**
The serialization format for messages on the wire. The protocol defines both Protobuf and JSON schemas. Implementations choose one.
