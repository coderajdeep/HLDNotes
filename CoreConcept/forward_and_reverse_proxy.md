This is one of the most important networking topics for **System Design interviews**. Many people confuse **Forward Proxy, Reverse Proxy, VPN, and Load Balancer** because all of them sit between two parties. The key difference is **who they represent and what problem they solve**.

---

# 1. Forward Proxy

A **Forward Proxy** sits **between clients and the internet**.

It represents the **client**.

```
                Internet
                    ^
                    |
            +----------------+
            | Forward Proxy  |
            +----------------+
               ^   ^    ^
              /    |     \
        Client1 Client2 Client3
```

Instead of connecting directly to Google,

```
Client ---> Google
```

the request becomes

```
Client ---> Forward Proxy ---> Google
```

Google sees the proxy's IP, **not the client's IP**.

---

## Why use it?

* Hide client IP
* Control internet access
* Cache responses
* Log outgoing requests
* Block websites

---

## Example

Imagine a company.

Employees are not allowed to browse every website.

```
Employee
     |
     v
Forward Proxy
     |
     +----> Google ✓
     |
     +----> YouTube ✗
     |
     +----> Facebook ✗
```

The proxy checks company policies before forwarding requests.

---

## Another example

A scraper making millions of requests.

Without proxy:

```
Scraper ---> Amazon
```

Amazon blocks the IP.

With proxies:

```
Scraper
   |
   +--> Proxy1 --> Amazon
   |
   +--> Proxy2 --> Amazon
   |
   +--> Proxy3 --> Amazon
```

Every request appears to come from different IPs.

---

# 2. Reverse Proxy

A **Reverse Proxy** sits **between the internet and your servers**.

It represents the **servers**.

```
              Internet
                  |
                  v
          +----------------+
          | Reverse Proxy  |
          +----------------+
             |    |    |
             v    v    v
          App1 App2 App3
```

Clients never directly access backend servers.

Instead,

```
Client
   |
Reverse Proxy
   |
Backend Server
```

---

## Why use it?

* Hide backend servers
* SSL termination
* Authentication
* Caching
* Compression
* Routing
* Load balancing

---

## Example

Suppose Netflix has 500 servers.

Users should never know their addresses.

```
User

     |

Reverse Proxy

     |

+-------+-------+-------+
|       |       |       |
App1   App2   App3
```

The reverse proxy chooses which server should handle the request.

---

# Real Example

When you visit

```
https://www.netflix.com
```

Actually,

```
Browser
    |
Cloudflare / NGINX
    |
Netflix Servers
```

The browser never talks directly to Netflix servers.

---

# Client Visibility

Forward Proxy

```
Client knows proxy exists.

Client
   |
Forward Proxy
   |
Internet
```

Reverse Proxy

```
Client usually doesn't know it exists.

Client
   |
Reverse Proxy
   |
Server
```

---

# Forward vs Reverse Proxy

| Feature         | Forward Proxy       | Reverse Proxy    |
| --------------- | ------------------- | ---------------- |
| Represents      | Client              | Server           |
| Hides           | Client              | Server           |
| Used by         | Client organization | Server owner     |
| Controls        | Outgoing traffic    | Incoming traffic |
| Internet sees   | Proxy IP            | Proxy IP         |
| Backend visible | Yes                 | No               |

---

# 3. Proxy vs VPN

People often think they are the same.

They are not.

---

## Proxy

Only the application configured to use the proxy sends traffic through it.

```
Browser
   |
Proxy
   |
Internet
```

Other apps are unaffected.

---

## VPN

A VPN creates an encrypted tunnel.

```
Laptop
    |
Encrypted Tunnel
    |
VPN Server
    |
Internet
```

All traffic goes through the VPN.

* Browser
* Spotify
* Games
* Terminal
* APIs

Everything.

---

## Example

Without VPN

```
Laptop
   |
ISP
   |
Google
```

ISP can see destination websites.

With VPN

```
Laptop
    |
Encrypted Tunnel
    |
VPN Server
    |
Google
```

ISP sees only encrypted traffic to the VPN server.

---

## Proxy vs VPN

| Feature             | Proxy      | VPN                               |
| ------------------- | ---------- | --------------------------------- |
| Encrypts traffic    | Usually no | Yes                               |
| Hides IP            | Yes        | Yes                               |
| Covers whole device | No         | Yes                               |
| Faster              | Usually    | Slightly slower due to encryption |
| Security            | Low        | High                              |

---

# 4. Reverse Proxy vs Load Balancer

These are related but not identical.

Many reverse proxies can also perform load balancing.

---

## Reverse Proxy

Primary purpose:

```
Hide backend servers
```

```
Internet

     |

Reverse Proxy

     |

Single Server
```

Even with one backend server, a reverse proxy is useful.

---

## Load Balancer

Primary purpose:

```
Distribute traffic
```

```
          Client
             |
      Load Balancer
      /     |      \
     /      |       \
 App1     App2     App3
```

It decides:

* Round Robin
* Least Connections
* Weighted
* IP Hash

---

## Example

1000 users

Without load balancer

```
1000 Users

      |

Server1

(Overloaded)
```

With load balancer

```
1000 Users

      |

Load Balancer

  /    |     \

S1    S2    S3
```

Each gets roughly one-third of the requests.

---

# Can Reverse Proxy be a Load Balancer?

Yes.

For example:

* NGINX
* HAProxy
* Envoy

can do both:

```
                Internet
                     |
             Reverse Proxy
                     |
            Load Balancing
          /       |       \
       App1     App2     App3
```

---

# Can Load Balancer be a Reverse Proxy?

In many practical deployments, yes.

For example:

* AWS Application Load Balancer (ALB)
* Google Cloud Load Balancer
* Azure Application Gateway

act as reverse proxies because clients connect to them instead of directly to backend servers.

However, conceptually, **load balancing is a function**, while **reverse proxy describes the position and role of representing backend servers**.

---

# Proxy vs Load Balancer

| Feature                | Proxy                       | Load Balancer           |
| ---------------------- | --------------------------- | ----------------------- |
| Main purpose           | Intermediary                | Distribute traffic      |
| Client-side            | Forward proxy               | No                      |
| Server-side            | Reverse proxy               | Yes                     |
| Chooses backend server | Usually not (forward proxy) | Yes                     |
| Supports caching       | Yes                         | Sometimes               |
| SSL termination        | Reverse proxy               | Yes (L7 load balancers) |

---

# Real Production Example

Suppose you're accessing an e-commerce application.

```
                Internet
                     |
             DNS (shop.com)
                     |
             Reverse Proxy / CDN
           (Cloudflare, NGINX)
                     |
          Application Load Balancer
                     |
          +----------+----------+
          |          |          |
        App1       App2       App3
          |
       Redis / Database
```

Flow:

1. Browser sends request to `shop.com`.
2. The reverse proxy handles SSL termination, security checks, caching, and request routing.
3. The load balancer distributes traffic across healthy application instances.
4. The selected application server processes the request and talks to Redis or the database.
5. The response travels back through the load balancer and reverse proxy to the client.

---

# Interview Summary

| Component         | Represents      | Used By                  | Primary Purpose                                                                   |
| ----------------- | --------------- | ------------------------ | --------------------------------------------------------------------------------- |
| **Forward Proxy** | Client          | Client organization      | Hide/control client access to external services                                   |
| **Reverse Proxy** | Server          | Server owner             | Protect, optimize, and route incoming requests to backend servers                 |
| **VPN**           | Client device   | End user or organization | Encrypt and tunnel network traffic while hiding the client's IP                   |
| **Load Balancer** | Backend service | Server owner             | Distribute requests across multiple servers for scalability and high availability |

### A simple way to remember

* **Forward Proxy** → **Clients → Proxy → Internet** ("Go out on my behalf.")
* **Reverse Proxy** → **Internet → Proxy → Servers** ("Receive requests on our behalf.")
* **VPN** → **Secure encrypted tunnel** for device traffic.
* **Load Balancer** → **One entry point, many backend servers**. It focuses on spreading traffic efficiently rather than simply acting as an intermediary.
