# Cache Architecture Patterns

Here's a rundown of the main patterns, organized by how they handle reads and writes.

## Read Patterns

**Cache-Aside (Lazy Loading)**

The application checks the cache first; on a miss, it reads from the database and populates the cache itself.
- Pros: Only requested data gets cached; resilient to cache failures (falls back to DB)
- Cons: First request always misses (cold start penalty); risk of stale data if the DB changes elsewhere
- Common in: Redis/Memcached setups fronting relational databases

**Read-Through**

Similar to cache-aside, but the cache itself owns the logic for loading from the database, the application always talks to the cache, never the DB directly. 
CDNs are a form of read-through cache. When a CDN gets a cache miss, it fetches from your origin server, caches the result, and returns it. But for application-level caching with Redis, cache-aside is far more common.
- Pros: Simpler application code; consistent loading logic
- Cons: Requires a caching layer that supports this (e.g. some managed caches); still has cold-start misses.

## Write Patterns

**Write-Through**

Every write goes to the cache and the database synchronously, as a single operation. The tradeoff is slower writes and Write-through can also pollute the cache with data that may never be read again.
Write-through still suffers from the dual-write problem. If the cache update succeeds but the database write fails, or vice versa, the systems can end up inconsistent. Use this when reads must always return fresh data and your system can tolerate slightly slower writes.
- Pros: Cache is always consistent with the DB
- Cons: Higher write latency (waiting on two systems); you cache data that might never be read

**Write-Around**

Writes go directly to the database, bypassing the cache. The cache only gets populated on subsequent reads (cache-aside style).
- Pros: Avoids flooding the cache with write-heavy data that isn't read often
- Cons: Recently written data isn't in cache, so the first read after a write is slow

**Write-Behind (Write-Back)**

Writes go to the cache first and are asynchronously flushed to the database later, often batched.
Use this when you need high write throughput and eventual consistency is acceptable. Common in analytics and metrics pipelines.
- Pros: Very low write latency; can batch/reduce DB load
- Cons: Risk of data loss if the cache crashes before flushing; more complex to implement correctly (ordering, retries)

## Structural Patterns

**Cache Distribution**

- *Client-side/local cache*: In-process memory (e.g., a Java HashMap, Python dict). Fastest possible access, but not shared across instances and can go stale.
- *Distributed cache*: Separate service (Redis, Memcached) shared across app instances. Consistent view, but adds network hop latency.
- *CDN/edge cache*: Caches static or semi-static content geographically close to users.

**Multi-level (Tiered) Caching**

Combines local (L1) and distributed (L2) caches — check local memory first, then a shared cache, then the database. Common in high-throughput systems where even a network call to Redis is too slow for hot-path data.

**Cache Invalidation Strategies** (the "hard problem" alongside naming things)
- TTL (time-based expiration) — simple, but can serve stale data or cause "thundering herd" refresh spikes
- Event-based invalidation — the write path explicitly evicts or updates related keys
- Versioned/keyed cache entries — e.g., embedding a version number in the key so old entries just become irrelevant rather than needing explicit deletion

## Choosing Between Them

A few questions that usually drive the decision:
- **Read/write ratio**: Read-heavy → cache-aside or read-through. Write-heavy → write-behind or write-around.
- **Consistency tolerance**: Can you tolerate slightly stale reads? If yes, you have more flexibility (TTLs, async writes). If not, write-through with careful invalidation.
- **Failure mode**: What happens if the cache goes down? Cache-aside degrades gracefully (falls back to DB); write-behind risks data loss.
- **Data access pattern**: Uniformly hot data benefits from tiered caching; highly skewed "hot key" access might need sharding or replication of specific keys.

Want me to go deeper on any of these — like how thundering herd or cache stampede problems get mitigated, or how this plays out in a specific system you're designing?
