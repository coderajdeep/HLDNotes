Yes, **an ISP can inspect Layer 7 data in some cases, but not always**. It depends on whether the traffic is encrypted.

Let's break it down.

## 1. HTTP (No Encryption)

If you're using plain HTTP:

```text
Browser
    |
HTTP Request
    |
ISP
    |
Website
```

The ISP can see everything:

* ✅ URL (e.g., `http://example.com/products`)
* ✅ HTTP headers
* ✅ Cookies
* ✅ Request body
* ✅ Response body
* ✅ Images, HTML, JSON

Because HTTP is sent in plaintext.

---

## 2. HTTPS (TLS Encryption)

Today, almost all websites use HTTPS.

```text
Browser
    |
TLS Encrypted Traffic
    |
ISP
    |
Website
```

The ISP **cannot** read the encrypted Layer 7 payload.

It **cannot** see:

* ❌ URL path (`/products/123`)
* ❌ HTTP headers
* ❌ Cookies
* ❌ Request body
* ❌ Response body

However, it can still observe some metadata.

### What the ISP can still see

* ✅ Your IP address
* ✅ The destination IP address
* ✅ The port (usually 443)
* ✅ How much data is transferred
* ✅ Timing of packets
* ✅ The domain name in many cases

The last point deserves more explanation.

---

## 3. How does the ISP know the website?

There are two common ways:

### DNS lookup

Before connecting, your device usually asks a DNS server:

```text
www.google.com -> 142.250.x.x
```

If you're using your ISP's DNS server, the ISP sees this lookup.

---

### TLS SNI (Server Name Indication)

During the TLS handshake, the client often sends the hostname so the server knows which certificate to present.

Historically, this **SNI** was sent in plaintext, allowing the ISP to see:

```text
Host: www.google.com
```

even though the HTTP request itself remained encrypted.

Today, newer protocols such as **Encrypted Client Hello (ECH)** aim to encrypt this information, but ECH is not yet universally deployed.

---

## 4. What changes with a VPN?

```text
Browser
    |
Encrypted VPN Tunnel
    |
VPN Server
    |
Website
```

The ISP can see:

* ✅ Your IP
* ✅ The VPN server's IP
* ✅ The amount of traffic
* ✅ Timing information

The ISP **cannot** see:

* ❌ Which websites you're visiting (assuming no DNS leaks and the VPN is configured correctly)
* ❌ HTTP requests
* ❌ Layer 7 payload

---

## Summary

| Information            | HTTP  | HTTPS                                         | HTTPS + VPN                              |
| ---------------------- | ----- | --------------------------------------------- | ---------------------------------------- |
| Read HTTP payload      | ✅ Yes | ❌ No                                          | ❌ No                                     |
| See URL path           | ✅ Yes | ❌ No                                          | ❌ No                                     |
| See destination domain | ✅ Yes | Usually yes (via DNS or SNI/ECH availability) | Usually no (VPN hides it from the ISP)   |
| See destination IP     | ✅ Yes | ✅ Yes                                         | ❌ No (ISP sees only the VPN server's IP) |
| See packet size/timing | ✅ Yes | ✅ Yes                                         | ✅ Yes                                    |

### Interview takeaway

A common interview question is:

> **Can an ISP inspect Layer 7 packets?**

A good answer is:

> **For HTTP, yes—because the Layer 7 data is unencrypted. For HTTPS, the Layer 7 payload is encrypted by TLS, so the ISP cannot inspect the application data. The ISP can still observe metadata such as IP addresses, ports, packet sizes, timing, and often the destination domain via DNS or the TLS handshake (unless technologies like encrypted DNS and ECH are in use).**
