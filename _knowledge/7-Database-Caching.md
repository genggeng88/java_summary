---
layout: default
title: Database Caching
topic: Topic 7
---

# Database Caching Concepts

**TTL**: Time To Live is a setting defines how long a cached item remains valid before it expires. After this time, the cached data is considered stale, and subsequent requests	will fetch fresh data, typically from the underlying data source. After TTL, entry becomes expired and any further requests will result in a cache miss, prompting a fresh database call.

**Cache Hit**: when the requested data is found in the cache, and it is served without querying the database or other data sources.

**Cache Miss**: when the requested data is not found in the cache, and the data must be retrieved from the original data source. After this, the data can be added to cache for further requests. 

**Eviction Policies**: Evict policies determine how cache entries are replaced when the cache reaches its maximum size. Adding a TTL ensures that even when cache capacity is not reached, stale entries are removed. Common eviction policies include: 

1. **LRU + TTL**: Least Recently Used + TTL
    1. LRU: This eviction policy evicts the least recently accessed 	data from the cache when the cache reaches its capacity. 
    2. TTL: Data is automatically removed when its TTL expires, even if it’s frequently accessed. 

2. **LFU + TTL**: Least Frequently Used + TTL
    1. LFU: evicts entries that are accessed the least number of times.

3. **FIFO + TTL**: First In, First Out + TTL
    1. FIFO: evicts the oldest data first, regardless of how frequently it is accessed.

Common Issues in Caching:

**Cache Avalanche**: cache avalanche happens when multiple cache entries expire simultaneously, leading to a surge in requests to the database. This often happens when caches for frequently requested data expire at the same time, resulting in an overload on the backend database or service. The sudden flood of database requests can lead to performance degradation, slow response times, or even system failure due to database overload.

**Solution:** Random Window for TTL: Introduce randomness to the TTL value so that cache entries expire at slightly different times. For instance, instead go setting a fixed TTL of 30 minutes, we can randomize it by adding a small random value (TTL = 30 MIN + random.nextInt(20) SECONDS). This spreads out expiration times, reducing the chance of simultaneous expirations.


**Cache Penetration:** cache penetration occurs when requests for non-existent data bypass the cache and directly hit the database. This can result in frequent cache misses for the same non-existent data, causing unnecessary database load. Frequent requests for data that don’t exist lead to repeated cache misses and stress the database, increasing latency and response times.

**Solution:** Cache Null Values that we can cache the result of a “null” response (e.g., a 404 or “not found” result). By caching the null response, future requests for the same non-existent data will hit the cache and avoid querying the database. 