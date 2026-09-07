## Load Balancing

**Load balancing** is the process of distributing incoming network requests across multiple servers so that no single server becomes overloaded.

For example, suppose you have 3 backend servers:

```text
                 ┌─── Server 1
Users ──► Load Balancer ─── Server 2
                 └─── Server 3
```

Instead of sending all requests to one server, the load balancer distributes them across the available servers.

### Why do we need it?

1. **High availability** — if one server fails, traffic can go to other servers.
2. **Scalability** — you can add more servers as traffic increases.
3. **Better performance** — requests are distributed instead of concentrating on one server.
4. **Fault tolerance** — unhealthy servers can be removed from traffic.
5. **Reduced response time** — users can be routed to an appropriate/nearby server.

---

# Types of Load Balancing

There are several ways to classify load balancing. The most important distinction is **where the load balancing happens**.

## 1. Layer 4 Load Balancing

Layer 4 operates at the **Transport Layer** of the OSI model.

It primarily uses:

- Source IP
- Destination IP
- Source port
- Destination port
- TCP/UDP protocol

It does **not need to understand HTTP requests**.

```text
Client
   │
   ▼
L4 Load Balancer
   │
   ├──► Server 1
   ├──► Server 2
   └──► Server 3
```

### Example

A TCP connection comes to:

```text
10.0.0.10:443
```

The load balancer decides which backend server should receive the connection.

### Advantages

- Very fast
- Low overhead
- Good for very high traffic
- Works with TCP/UDP applications

### Examples

- AWS Network Load Balancer (NLB)
- HAProxy in TCP mode
- Linux IPVS
- Azure Load Balancer

---

# 2. Layer 7 Load Balancing

Layer 7 operates at the **Application Layer**.

It understands protocols such as HTTP/HTTPS and can make routing decisions based on:

- URL
- HTTP method
- Headers
- Cookies
- Hostname
- Query parameters

For example:

```text
example.com/users/*  ──► User Service
example.com/orders/* ──► Order Service
example.com/payment/* ──► Payment Service
```

The load balancer actually understands the HTTP request.

### Example

```http
GET /orders/123
Host: example.com
```

The load balancer can inspect `/orders/123` and route it to the appropriate backend.

### Advantages

- Intelligent routing
- URL-based routing
- Host-based routing
- Can perform TLS termination
- Can inspect HTTP headers/cookies

### Examples

- AWS Application Load Balancer (ALB)
- NGINX
- HAProxy in HTTP mode
- Envoy
- Apache HTTP Server

---

# 3. DNS Load Balancing

Here the **DNS system** distributes users among multiple IP addresses.

For example:

```text
api.example.com
       │
       ▼
     DNS
   /   |   \
  /    |    \
IP1   IP2   IP3
```

DNS might return:

```text
IP1 → Server 1
IP2 → Server 2
IP3 → Server 3
```

This is commonly used for distributing traffic across different geographic locations.

### Example

```text
India user      → India server
US user         → US server
Europe user     → Europe server
```

Examples:

- AWS Route 53
- Cloudflare DNS
- Google Cloud DNS

**Important:** DNS load balancing is different from a traditional load balancer. DNS generally doesn't see individual HTTP requests; it controls which IP address a client resolves to.

---

# 4. Global Server Load Balancing (GSLB)

GSLB distributes traffic across **multiple data centers or geographic regions**.

For example:

```text
                 Global Load Balancer
                   /      |       \
                  /       |        \
             US Region  EU Region  India Region
                │          │          │
             Servers     Servers    Servers
```

It can consider factors such as:

- Geographic location
- Latency
- Data-center health
- Availability
- Traffic load

This is particularly useful for globally distributed applications.

---

# Load Balancing Algorithms

Once a load balancer receives a request/connection, it needs an algorithm to decide:

> **Which backend server should handle this request?**

There are several common algorithms.

---

## 1. Round Robin

The simplest algorithm.

Requests are distributed sequentially:

```text
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
Request 5 → Server 2
Request 6 → Server 3
```

### Example

```text
S1 → S2 → S3 → S1 → S2 → S3
```

### Good when

Servers have approximately equal capacity and requests have similar processing requirements.

---

# 2. Weighted Round Robin

Each server gets a **weight**.

For example:

```text
Server 1 → Weight 5
Server 2 → Weight 3
Server 3 → Weight 2
```

Approximately:

```text
S1 S1 S1 S1 S1
S2 S2 S2
S3 S3
```

Server 1 receives more traffic because it has greater capacity.

---

# 3. Least Connections

The load balancer sends the new request to the server with the **fewest active connections**.

Example:

```text
Server 1 → 20 connections
Server 2 → 8 connections
Server 3 → 15 connections
```

New request:

```text
             ↓
        Load Balancer
             ↓
         Server 2
```

This is useful when requests can have significantly different processing times.

---

# 4. Weighted Least Connections

This combines:

- Number of active connections
- Server capacity/weight

For example:

```text
Server 1 → powerful → weight 5
Server 2 → medium   → weight 3
Server 3 → small    → weight 1
```

The algorithm considers both the current connections and server capacity.

---

# 5. IP Hash

The load balancer calculates a hash based on the client's IP address.

Conceptually:

```text
hash(client IP) % number_of_servers
```

For example:

```text
Client A → hash → Server 1
Client B → hash → Server 3
Client C → hash → Server 2
```

The same client IP will generally be routed to the same server as long as the server set remains stable.

### Useful for

**Session persistence / sticky sessions.**

However, IP hashing can produce uneven distribution if many clients share the same IP, such as users behind a corporate NAT.

---

# 6. Consistent Hashing

Consistent hashing is commonly used when you want requests/keys to remain associated with particular servers even when servers are added or removed.

It is particularly common in:

- Distributed caches
- Distributed databases
- Key-value systems

Example:

```text
              Hash Ring

          S1
       /      \
     S4        S2
       \      /
          S3
```

A key is hashed onto the ring and assigned to the next appropriate server.

One major advantage is that **adding/removing a server doesn't require remapping every key**.

---

# 7. Least Response Time

The load balancer considers server response performance and routes traffic toward servers responding more quickly.

For example:

```text
Server 1 → 100 ms
Server 2 → 40 ms
Server 3 → 80 ms
```

New traffic is more likely to go to Server 2.

This can be useful when backend servers have different performance characteristics.

---

# 8. Random

The load balancer randomly selects a backend server.

```text
Request → Random selection → Server 2
Request → Random selection → Server 1
Request → Random selection → Server 3
```

Simple, but generally less predictable than algorithms such as least connections or round robin.

---

# Real-World Load Balancers

| Load Balancer | Type | Common algorithms/features |
|---|---|---|
| **AWS ALB** | L7 | Round robin, least outstanding requests |
| **AWS NLB** | L4 | Flow-based distribution |
| **NGINX** | L4/L7 | Round robin, weighted RR, least connections, IP hash |
| **HAProxy** | L4/L7 | Round robin, least connections, hashing, weights |
| **Envoy** | L4/L7 | Round robin, least request, ring hash, random |
| **Azure Load Balancer** | L4 | Hash-based distribution |
| **Google Cloud Load Balancing** | L4/L7 | Multiple routing/distribution mechanisms |
| **Cloudflare Load Balancing** | L7/global | Geographic, latency, health-based routing |

---

# Putting It Together

Consider an e-commerce application:

```text
                         Internet
                            │
                            ▼
                   ┌─────────────────┐
                   │  Load Balancer   │
                   │    L7 / HTTP     │
                   └────────┬────────┘
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
        ┌─────────┐    ┌─────────┐    ┌─────────┐
        │ Server 1│    │ Server 2│    │ Server 3│
        │  API    │    │  API    │    │  API    │
        └─────────┘    └─────────┘    └─────────┘
```

Suppose Server 1 currently has:

```text
20 connections
```

Server 2:

```text
5 connections
```

Server 3:

```text
12 connections
```

With **Least Connections**, the next request/connection will preferentially go to:

```text
             Load Balancer
                   │
                   ▼
              Server 2
```

If instead you use **Round Robin**:

```text
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
...
```

---

## One important distinction

You can think about load balancing at **two different levels**:

```text
                 Load Balancing
                       │
          ┌────────────┴────────────┐
          │                         │
       Where?                    How?
          │                         │
     L4 / L7 / DNS          Round Robin
     / Global LB            Least Connections
                            Weighted RR
                            IP Hash
                            Consistent Hash
                            etc.
```

So **"L4 vs L7" and "Round Robin vs Least Connections" are not competing classifications**.

For example:

> **NGINX L7 Load Balancer + Least Connections algorithm**

is a perfectly valid combination.

And:

> **AWS ALB + L7 + Round Robin**

is another example.

### For system-design interviews

The most important concepts to understand deeply are:

**L4 vs L7 → algorithms → health checks → sticky sessions → session/state management → connection draining → TLS termination → horizontal scaling → failover → DNS/GSLB.**

These concepts together form the foundation of how load balancing is used in real distributed systems.
