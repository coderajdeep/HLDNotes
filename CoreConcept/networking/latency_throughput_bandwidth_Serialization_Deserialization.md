These four concepts are closely related in **computer networks and distributed systems**. A simple way to think about them is:

> **Bandwidth = how much data the network can carry**  
> **Latency = how long one piece of data takes to reach its destination**  
> **Throughput = how much data is actually transferred per unit of time**  
> **Serialization/Deserialization = converting data between in-memory objects and a transferable format**

---

## 1. Latency

**Latency** is the **time taken for data to travel from one point to another**.

Usually measured in **milliseconds (ms)**.

### Example

Suppose:

```text
Client ───────────────> Server
          50 ms
```

If the server receives your request 50 ms after you send it, the network latency is approximately **50 ms** one way.

For an HTTP request, the overall response time may include:

```text
Client
  |
  | Request
  |---------> Server
  |             |
  |             | Processing
  |             |
  |<---------   |
  | Response
```

The user-visible time is affected by:

- Network latency
- Server processing time
- Database latency
- Serialization/deserialization
- Network transfer time

### Simple analogy

Think of a highway.

**Latency = time taken for one car to travel from A to B.**

---

# 2. Bandwidth

**Bandwidth** is the **maximum amount of data that a network connection can theoretically transmit per unit of time**.

Usually measured in:

- Mbps = megabits/second
- Gbps = gigabits/second

For example:

```text
Network connection = 1 Gbps
```

means the connection can theoretically carry up to approximately **1 gigabit of data per second**.

### Analogy

Think of a highway again.

**Bandwidth = number of lanes on the highway.**

A highway with 10 lanes can carry more cars simultaneously than a highway with 2 lanes.

Similarly:

```text
100 Mbps  <  1 Gbps  <  10 Gbps
```

means increasing network capacity.

---

# 3. Throughput

**Throughput** is the **actual amount of data successfully transferred per unit of time**.

For example, suppose you have:

```text
Bandwidth = 1 Gbps
```

but due to network congestion, protocol overhead, packet loss, server limitations, etc., you're actually transferring:

```text
Throughput = 600 Mbps
```

So:

> **Bandwidth is the capacity; throughput is the actual achieved rate.**

### Analogy

Highway:

```text
Bandwidth → How many lanes the highway has
Throughput → How many cars are actually passing per second
```

Even if the highway has 10 lanes, traffic congestion may mean only 7 lanes are effectively being used.

---

# 4. Latency vs Bandwidth vs Throughput

| Concept | Meaning | Typical unit |
|---|---|---|
| **Latency** | How long data takes to travel | ms |
| **Bandwidth** | Maximum data-carrying capacity | Mbps/Gbps |
| **Throughput** | Actual data transferred per second | Mbps/Gbps |

### Example

Imagine downloading a file:

```text
Server ─────────────────────> Client

Latency:     50 ms
Bandwidth:   1 Gbps
Throughput:  700 Mbps
```

Here:

- **50 ms** → how long it takes for data to travel
- **1 Gbps** → maximum capacity of the connection
- **700 Mbps** → actual transfer rate

---

# 5. Serialization

Now let's look at **serialization**.

A program usually works with data as objects/structures in memory.

For example, your application might have:

```text
User object

name = "Rajdeep"
age = 30
```

But if you want to send this data over a network, you need to convert it into a format that can be transmitted.

This conversion is called **serialization**.

```text
Object in memory
       ↓
  Serialization
       ↓
JSON / XML / Protobuf / etc.
       ↓
    Network
```

### Example

Application object:

```text
User
name = "Rajdeep"
age = 30
```

Serialized as JSON:

```json
{
  "name": "Rajdeep",
  "age": 30
}
```

That JSON can then be sent through HTTP.

---

# 6. Deserialization

**Deserialization** is the reverse process.

The server receives the serialized data and converts it back into a representation that the application can work with.

```text
Network
   ↓
JSON / Protobuf / etc.
   ↓
Deserialization
   ↓
Object / data structure in memory
```

For example:

```text
JSON
{
  "name": "Rajdeep",
  "age": 30
}
        ↓
Deserialization
        ↓
User object
name = "Rajdeep"
age = 30
```

---

# 7. Complete HTTP example

Suppose you call:

```text
POST /users
```

with:

```json
{
  "name": "Rajdeep",
  "age": 30
}
```

The process roughly looks like:

```text
Client Application
      |
      | User object
      ↓
 Serialization
      |
      | JSON
      ↓
 HTTP
      |
      ↓
 Network
      |
      ↓
 Server
      |
      ↓
 Deserialization
      |
      | User object
      ↓
 Server Application
```

When the server sends a response, the reverse happens:

```text
Server object
      ↓
 Serialization
      ↓
 JSON
      ↓
 Network
      ↓
 Client
      ↓
 Deserialization
      ↓
 Client object
```

---

## 8. How they are connected

Imagine you're sending a **1 GB file** over a network.

Three important factors are:

```text
             Network
                |
       ┌────────┼────────┐
       ↓        ↓        ↓
    Latency  Bandwidth Throughput
```

And before transmission:

```text
Application data
      ↓
Serialization
      ↓
Network transmission
      ↓
Deserialization
      ↓
Application data
```

### One important distinction

**Bandwidth does not necessarily determine how fast a single request completes.**

For example, you could have:

```text
Bandwidth = 10 Gbps
Latency   = 200 ms
```

The network has enormous capacity, but each round trip can still take 200 ms.

Conversely:

```text
Bandwidth = 10 Mbps
Latency   = 1 ms
```

The network responds very quickly, but transferring a large amount of data will be slow.

So, in distributed systems:

> **Latency matters especially for request/response operations and small messages, while bandwidth/throughput matter especially for large amounts of data.**
