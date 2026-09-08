### SSE — Short Note

- **SSE (Server-Sent Events)** is a mechanism for **server → client** real-time communication over a **long-lived HTTP connection**.
- The SSE connection is **stateful at the connection level**: the client remains connected to one server.
- A **Layer-7 HTTP/HTTPS Load Balancer** can be used. It must support **long-lived connections** and appropriate **timeouts**.
- **Sticky sessions** may be needed when SSE connection/subscription state is stored locally on a server.

### Shared Event Streaming

For scalability, use a shared event system such as **Redis Pub/Sub, Redis Streams, or Kafka**:

```text
Client → Load Balancer → Server A
                           │
                           ▼
                    Kafka / Redis
                           ▲
                           │
                    Server B / C
```

- The **SSE connection remains with one server**; the shared system distributes events between servers.
- **Redis Pub/Sub:** real-time but **not durable**. If a server is down or disconnected, events can be missed.
- **Redis Streams / Kafka:** **durable event streaming**; events can be retained and consumed/replayed after reconnection or server failure.
- If Server A fails, the client’s SSE connection breaks → `EventSource` reconnects → Load Balancer can send it to Server B.
- Server B can use the **shared event stream** to continue delivering relevant events.

**Key takeaway:**  
> **SSE = long-lived server → client connection; Load Balancer = distributes connections; Kafka/Redis Streams = shared, durable event stream for scalable and reliable event delivery.**

[ChatGPT Link](https://chatgpt.com/share/6a9fd470-f9e0-83e8-b48c-9b1b9020c4ea)
