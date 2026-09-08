## WebRTC — Web Real-Time Communication

**WebRTC (Web Real-Time Communication)** enables real-time communication between browsers/applications, mainly for:

- 🎥 Video calls
- 🎙️ Voice calls
- 📺 Screen sharing
- 📁 Peer-to-peer data transfer

### How WebRTC works

```text
User A                         User B
  │                              │
  │──── Signaling ──────────────>│
  │<──── SDP / ICE ─────────────>│
  │                              │
  │══════ Direct WebRTC ═════════│
  │     Audio / Video / Data      │
```

1. **Signaling** – Exchanges connection information between peers, such as **SDP (Session Description Protocol)** and **ICE (Interactive Connectivity Establishment)** candidates. WebRTC does not define the signaling mechanism; applications can use **HTTP (Hypertext Transfer Protocol)**, **WebSocket**, etc.

2. **ICE (Interactive Connectivity Establishment)** – Finds the best possible network path between the two peers, including paths through **NAT (Network Address Translation)** and firewalls.

3. **STUN (Session Traversal Utilities for NAT)** – Helps a peer discover its public-facing network address.

4. **TURN (Traversal Using Relays around NAT)** – Relays traffic through a server when a direct peer-to-peer connection cannot be established.

5. **Peer-to-peer communication** – After connection establishment, audio, video, or data can usually flow directly between the peers.

### Important WebRTC components

- **`RTCPeerConnection`** → Manages the WebRTC connection.
- **`getUserMedia()`** → Accesses the camera and microphone.
- **RTCDataChannel** → Enables peer-to-peer arbitrary data transfer.
- **STUN (Session Traversal Utilities for NAT)** → Discovers public network information.
- **TURN (Traversal Using Relays around NAT)** → Provides a relay when direct communication fails.
- **SDP (Session Description Protocol)** → Describes media capabilities and connection information.
- **ICE (Interactive Connectivity Establishment)** → Finds and selects a viable network path.

### WebRTC vs WebSocket

| | WebRTC | WebSocket |
|---|---|---|
| Main use | Real-time audio/video/data | Real-time messaging |
| Connection | Usually peer-to-peer | Client ↔ Server |
| Server required for data | Not necessarily | Yes |
| Audio/Video | Native support | Requires additional handling |
| Typical latency | Very low | Low |

**In short:** WebRTC provides the mechanisms for **real-time peer-to-peer communication**, while **Signaling, SDP, ICE, STUN, and TURN** help establish and maintain the connection.

[ChatGPT](https://chatgpt.com/share/6a9ff28a-c25c-83ee-8ba0-29b4191456a0)
