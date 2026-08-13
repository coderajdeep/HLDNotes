Hot keys are one of the classic pain points in Redis-backed systems — a single key gets read so often it saturates one shard's CPU/network while the rest of the cluster sits idle. Here's how to approach it, from detection to mitigation.

## 1. Detect them first

- `redis-cli --hotkeys` (works when `maxmemory-policy` is set to an LFU policy like `allkeys-lfu`) samples keys and reports access frequency.
- Avoid `MONITOR` in production — it dumps every command and adds real overhead.
- Better: instrument at the client/proxy layer. Log key access counts in your application or sidecar (e.g., a rolling counter per key, flushed periodically) so you get hot-key visibility without touching Redis's own performance.
- Cloud-managed Redis (ElastiCache, Redis Enterprise, GCP Memorystore) often exposes hot-shard/hot-key metrics natively — check those dashboards first.

## 2. The most effective fix: client-side (L1) caching

For read-heavy workloads, this is usually the biggest lever. Cache the hot key's value **in-process** on each application server for a short TTL (hundreds of ms to a few seconds), so most reads never hit Redis at all.

- Use a bounded local cache (e.g., Caffeine, an LRU map, `functools.lru_cache` with TTL wrapper) sized for just the hottest keys.
- Invalidate via Redis pub/sub or a version-bump key when the underlying value changes, or just rely on short TTL + eventual consistency if staleness is tolerable.
- This turns an O(requests) load on Redis into O(app-servers) load, which is often a 100–1000x reduction.

## 3. Key splitting / sharding a single hot key

If a key can't be cached locally (e.g., a counter that must stay near-real-time), split it across N copies:

- `key:0`, `key:1`, ... `key:N-1`, written/read by hashing the client or randomly picking a shard.
- For reads: pick a random shard, or read all N and merge (works well for counters — sum them).
- For counters specifically, consider a "sharded counter" pattern: increments go to a random shard, periodic aggregation reads all shards.

## 4. Request coalescing (single-flight)

If many app instances all need to refresh the same key simultaneously (e.g., after a TTL expiry causes a stampede), use a single-flight/dedup pattern so only one request per instance actually hits Redis/DB while others wait on that in-flight result. Libraries like `golang.org/x/sync/singleflight` or an equivalent mutex-per-key pattern in your language work well.

## 5. Read replicas — helps, but has limits

Routing reads to replicas spreads load across more nodes, but a single hot key still lands on one node per replica (Redis doesn't shard within a key). It helps horizontally by adding more replicas of the *same* data, but doesn't fix the fundamental "one key = one node" bottleneck the way local caching or key-splitting does.

## 6. Proxy/edge layer caching

For data that's read across many services (not just one app), a shared caching proxy (Envoy, a custom sidecar, or CDN edge caching for HTTP-exposed data) can absorb hot-key traffic before it ever reaches Redis.

## 7. Guard against stampedes on expiry

Combine with:
- **Probabilistic early expiration**: refresh the cache slightly before TTL expires, with jitter, so not all instances expire at once.
- **Stale-while-revalidate**: serve the stale value while one request refreshes it in the background.

---

### Practical combo for most systems
1. Local in-process cache (short TTL) for anything genuinely hot.
2. Sharded keys for hot counters/aggregates that can't tolerate staleness.
3. Single-flight to prevent thundering herds on cache miss.
4. Monitoring via `--hotkeys` or app-level counters to catch new hot spots before they cause incidents.

If you want, I can sketch out a diagram of this multi-tier read path (client → L1 cache → Redis → DB), or go deeper into any one pattern (e.g., sharded counters or single-flight implementation in a specific language).
