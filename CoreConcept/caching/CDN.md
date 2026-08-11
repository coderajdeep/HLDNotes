## CDN (Content Delivery Network)

A **CDN** is a geographically distributed network of proxy servers that cache and serve content (static assets, and sometimes dynamic content) from locations physically closer to the end user, reducing latency and offloading traffic from origin servers.

### Why it matters in system design interviews

CDNs are a go-to answer whenever you're asked to scale a system that serves content globally. Bringing it up shows you're thinking about latency, load, and cost — not just correctness.

**Key points to mention:**

1. **What it caches**: Static content — images, videos, CSS, JS, HTML pages. Some CDNs also support dynamic content acceleration.
2. **How it works**: User requests hit the nearest **edge server** (Point of Presence, or PoP) instead of the origin server. If the content is cached there, it's served immediately (cache hit); otherwise, the CDN fetches it from origin, caches it, and serves it (cache miss).
3. **Push vs Pull CDN**:
   - **Pull**: CDN fetches content from origin on first request, caches it, serves subsequent requests from cache. Simpler, good for sites with less traffic or unpredictable content.
   - **Push**: You proactively upload content to the CDN. Better for sites with heavy, predictable traffic (e.g., video platforms).
4. **Benefits**:
   - Reduced latency (content served from nearby edge nodes)
   - Reduced load on origin servers
   - Better availability/resilience (origin outage doesn't necessarily take down cached content)
   - DDoS mitigation (absorbs traffic at the edge)
5. **Trade-offs to discuss**:
   - **Cache invalidation** — a classic hard problem. How do you update/expire content across all edge nodes? (TTLs, versioned URLs/cache-busting, purge APIs)
   - **Cost** — CDNs charge for bandwidth/requests
   - **Staleness** — users might briefly see outdated content

### How to bring it up in an interview

When designing something like an image-sharing app, video streaming service, or news site, say something like:

> "Since static assets like images/videos don't change often and are read-heavy, I'd put a CDN in front of them so users get low-latency access from the nearest edge location, and I'd take load off our origin/storage servers."

Then be ready to discuss **cache invalidation strategy** if pushed — that's usually the deeper follow-up interviewers ask.
