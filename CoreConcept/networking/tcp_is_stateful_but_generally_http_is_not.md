Not necessarily. The key is that **TCP connection state and HTTP request state are two different things**.

### 1. TCP is stateful

A TCP connection maintains state between the client and server.

For example:

```text
Client                     Server
  |                          |
  | ---- TCP connection ---> |
  |                          |
  | <--- established ------> |
  |                          |
  | ====== Data ==========> |
  | <====== ACK =========== |
```

Once the connection is established, both sides maintain information such as:

- Sequence numbers
- ACK numbers
- Connection state
- Flow-control information
- Retransmission information

So **TCP connection is stateful**.

---

### 2. HTTP request is different

An HTTP request is an **application-level message** sent over TCP.

For example:

```text
TCP Connection
      |
      +---- HTTP Request 1
      |
      +---- HTTP Response 1
      |
      +---- HTTP Request 2
      |
      +---- HTTP Response 2
```

The same TCP connection can carry multiple HTTP requests.

For example, with HTTP/1.1 keep-alive:

```text
Client                         Server

     TCP connection
           |
           |---- GET /users ---->|
           |<--- response -------|
           |
           |---- GET /orders --->|
           |<--- response -------|
           |
           |---- GET /products ->|
           |<--- response -------|
           |
           X---- TCP closed -----X
```

The **TCP connection remains stateful**, but each HTTP request is a separate request/response interaction.

### Important distinction

| | TCP Connection | HTTP Request |
|---|---|---|
| Layer | Transport | Application |
| Stateful? | **Yes** | Usually **stateless** |
| Lifetime | Can contain multiple requests | Individual request |
| Maintains connection state? | Yes | No inherent connection state |
| Example | One TCP connection | `GET /users` |

So the important idea for HLD is:

> **TCP is stateful because it maintains connection state. HTTP is generally stateless because each request contains the information needed to process that request, independent of previous requests.**

One TCP connection can therefore carry **many independent HTTP requests**.

Also, HTTP itself can have **application-level state** using cookies, sessions, authentication tokens, etc. That's separate from TCP's state.
