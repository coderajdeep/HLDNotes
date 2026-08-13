Choosing a shard key is one of those decisions that's hard to reverse later, so it's worth getting right up front. Here's the framework I'd use.

**The core properties you're balancing**

1. **High cardinality** — the key needs enough distinct values to actually spread data across all your shards. A boolean or a "status" field with 5 values will never let you scale past a handful of shards.

2. **Even distribution (no hotspots)** — even with high cardinality, values can cluster. If 80% of your rows have `country = "US"`, sharding on country alone gives you one enormous shard and several tiny ones.

3. **Query isolation** — ideally, most of your read queries include the shard key, so the database can route the query to one shard instead of fanning out to all of them (scatter-gather). Look at your actual query patterns, not just your data model.

4. **Write distribution** — this is the one people miss most often. A monotonically increasing key (auto-increment ID, timestamp, ULID) means all new writes land on the same shard at any given moment, creating a hot shard even though the data is technically "high cardinality" over time.

5. **Immutability** — the value shouldn't change after the record is written. If it does, you're paying for a cross-shard migration every time it's updated.

**Common patterns**

- **Hashed sharding**: hash the key to get uniform distribution. Solves hotspots well, but range queries ("all orders this week") now have to hit every shard, since sequential values no longer live near each other.
- **Range-based sharding**: keeps related values together, good for range scans, but risky if the underlying key is sequential (time-series data written in order will hammer one shard).
- **Compound key**: combine a low-cardinality field that matches your query pattern with a high-cardinality field for spread — e.g., `{tenant_id, user_id}` or `{device_id, timestamp}`. This is usually the sweet spot for real applications: you get query isolation on the first field and distribution from the second.

**Anti-patterns to avoid**

- Shard key = auto-increment ID or raw timestamp alone → hot shard on writes
- Shard key = low-cardinality field alone (status, boolean, region with only a few values) → uneven/limited shards
- Shard key on a field that changes often → expensive rebalancing

**A practical way to decide**

1. List your most frequent and most expensive queries — what's in the `WHERE` clause?
2. Estimate cardinality and skew of each candidate field against real (or representative) data, not assumptions.
3. Check your write pattern — will writes concentrate on one value at a time?
4. If one field alone doesn't satisfy both query isolation and distribution, go compound.
5. Simulate at your target scale if you can — skew that looks fine at 1M rows can look very different at 1B.

If you tell me your database (MongoDB, Cassandra, DynamoDB, Vitess/MySQL, etc.) and what your main query patterns look like, I can get more specific — the mechanics of "shard key" vs "partition key" vs "distribution key" differ enough across systems that it's worth tailoring.
