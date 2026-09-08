### HTTP Versions

- **HTTP/0.9:** Very basic; only `GET`, no headers, one request per TCP connection.
- **HTTP/1.0:** Added headers, status codes, methods like `POST`; connections were generally short-lived.
- **HTTP/1.1:** Persistent connections by default, `Host` header, chunked transfer, caching, etc. Still has HTTP-level **Head-of-Line (HOL) blocking**.
- **HTTP/2:** Binary framing, **multiplexing multiple streams over one TCP connection**, HPACK header compression. Client can send multiple requests, and the server can **respond to different streams independently and interleave their response data**—it does not wait for all requests to complete.
- **HTTP/3:** Uses **QUIC over UDP** instead of TCP, providing multiplexing without TCP-level HOL blocking and faster connection establishment.

**Evolution:**  
`HTTP/0.9 → HTTP/1.0 → HTTP/1.1 → HTTP/2 → HTTP/3`

**Key idea:** HTTP/2 introduced efficient **multiplexing**, where multiple request/response streams can progress simultaneously over a single connection.

[ChatGPT Link](https://chatgpt.com/share/6a9fc970-3414-83ee-91c5-650ed1c4b926)
