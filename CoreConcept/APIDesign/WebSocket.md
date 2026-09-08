## WebSocket

**WebSocket** is a communication protocol that provides a **persistent, full-duplex connection** between a client and server over a single TCP connection.

It is useful when both sides need to communicate **in real time**, such as chat, live notifications, multiplayer games, and live dashboards.

### How it works

1. Client first sends an HTTP request asking to **upgrade the connection to WebSocket**.
2. Server accepts the upgrade.
3. The connection becomes a persistent WebSocket connection.
4. Now **both client and server can send messages independently at any time**.

```text
Client                    Server
  |                         |
  |--- HTTP Upgrade ------->|
  |<-- 101 Switching -------|
  |                         |
  |<====== WebSocket ======>|
  |                         |
  |--- message ------------>|
  |<--- message ------------|
  |<--- message ------------|
  |--- message ------------>|
```

### WebSocket vs normal HTTP

| Feature | HTTP | WebSocket |
|---|---|---|
| Connection | Request/response | Persistent |
| Communication | Primarily client → server | Client ↔ Server |
| Server can initiate data | Not directly | Yes |
| Full-duplex | No | **Yes** |
| Real-time communication | Requires techniques like polling/SSE | **Native** |
| Typical use | REST APIs, web pages | Chat, gaming, live updates |

### Important point

WebSocket is **not just a faster HTTP connection**. HTTP is used for the initial handshake, but after the upgrade, communication uses the **WebSocket protocol** over the established TCP connection.

[ChatGPT link](https://chatgpt.com/share/6a9fecc1-5138-83ee-92e3-238c835455e1)

**In short:**

> **WebSocket = persistent + bidirectional + real-time communication between client and server.**
