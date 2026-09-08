### gRPC
**gRPC (Google Remote Procedure Call)** is a framework for communication between services, commonly used in **microservices**.

It uses **HTTP/2** for transport and **Protocol Buffers (Protobuf)** for defining and serializing messages.

- Uses **HTTP/2**
- Supports **request-response and streaming**
- Fast and efficient
- Uses **Protocol Buffers** by default

### Protocol Buffers (Protobuf)
**Protobuf** is a way to define and serialize structured data in a compact **binary format**.

Example:

```protobuf
message User {
    int32 id = 1;
    string name = 2;
}
```

It also acts as a **contract/schema** between client and server.

**In short:**  
> **gRPC = how services communicate**  
> **Protobuf = how the data/messages are defined and encoded**

[ChatGPT link](https://chatgpt.com/share/6a9fc685-8808-83e8-86b3-4b9df0805baa)
