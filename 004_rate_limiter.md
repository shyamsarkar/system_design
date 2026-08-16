# 🚦 Rate Limiter Design (Interview Ready)

## 🔹 1. Requirements

### Functional Requirements

* Limit requests by IP address, user, API key, or endpoint
* Allow different limits for different APIs and user plans
* Return a clear response when a limit is exceeded
* Support common algorithms: fixed window, sliding window, token bucket, and leaky bucket

### Non-Functional Requirements

* Low latency — the check should happen before business logic runs
* High availability — a limiter outage should not take down the API
* Distributed and consistent enough across multiple API servers
* Scalable for high request volume

---

## 🔹 2. Why Rate Limiting?

Rate limiting protects a service from:

* Abuse, spam, and brute-force attacks
* Accidental retry storms
* One client consuming all shared capacity
* Downstream provider quotas being exceeded

Example policy:

```text
100 requests per minute per API key
5 login attempts per minute per IP address
```

---

## 🔹 3. High-Level Architecture

```text
Client
  ↓
Load Balancer / API Gateway
  ↓
Rate Limiter ─────────────→ Redis (shared)
  ↓ allowed
┌──────────┴──────────┐
↓                     ↓
API Server 1          API Server 2
↓                     ↓
Application / Database
```

The rate limiter is commonly implemented at the API gateway, reverse proxy, or inside the API service. Redis is useful because increments and expiry operations are fast and shared by all API servers.

---

## 🔹 4. Request Flow

```text
Request arrives
      ↓
Identify client (API key, user ID, or IP)
      ↓
Build a rate-limit key
      ↓
Check and update counter in Redis atomically
      ↓
Under limit? ── Yes → Forward request to API
      │
      No
      ↓
Return HTTP 429 Too Many Requests
```

Useful response headers:

```text
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1710000060
Retry-After: 24
```

`X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` are the de-facto standard convention. `X-RateLimit-Reset` is the epoch time, in seconds, when the window resets.

### Retry Semantics

When a client receives `429`, it should honor `Retry-After` and retry with exponential backoff plus jitter to avoid a thundering herd. Idempotent requests can be retried safely; non-idempotent requests need care so a retry does not repeat a side effect.

### Back-of-the-Envelope Estimation

```text
Platform traffic:       10M requests/day
Active rate-limit keys: 100K
Memory per key:         ~100 bytes (counter + metadata)
Redis memory:           ~10 MB

Peak traffic:           100K requests/second
Redis round-trip:       ~0.1–0.5 ms
```

100K keys easily fit on one Redis node, but 100K checks per second can make one node the bottleneck. Use Redis Cluster or local caching when peak traffic requires more throughput.

---

## 🔹 5. Rate-Limit Key

The key defines what is being limited.

```text
rate_limit:{user_id}:{endpoint}
rate_limit:{api_key}
rate_limit:{ip_address}:login
```

Choose the identity carefully:

* **IP address** — useful before login, but users behind NAT can share an IP.
* **User ID** — fair for authenticated requests.
* **API key** — good for public or partner APIs.
* **Endpoint** — lets expensive endpoints have tighter limits.

---

## 🔹 6. Algorithms (🔥 Most Important)

### 1. Fixed Window Counter

Count requests in a fixed time interval.

```text
Key: user:42:12:00
Limit: 100 requests/minute
```

Implementation idea:

```text
SET rate_limit:user:42:12:00 0 NX EX 60
INCR rate_limit:user:42:12:00
```

The first command creates the counter with its TTL only when it does not exist; this avoids a successful `INCR` leaving a key without expiry if the process dies before a separate `EXPIRE`. Atomic setup matters because counters must always reset at the end of their window.

**Advantages:** simple, memory efficient, fast.

**Problem:** a client can send 100 requests at 12:00:59 and another 100 at 12:01:00. This boundary burst allows 200 requests in two seconds.

---

### 2. Sliding Window Log

Store a timestamp for every request and count timestamps in the last interval.

```text
Keep requests from now - 60 seconds to now
```

**Advantages:** accurate; no fixed-window boundary burst.

**Problem:** stores one entry per request, so memory and cleanup cost increase at high traffic.

---

### 3. Sliding Window Counter

Estimate the current count using the current and previous fixed windows.

```text
estimated_count = current_count + previous_count × (window_size − elapsed_in_current_window) / window_size
```

Worked example:

```text
Window size: 60 seconds
Elapsed in current window: 15 seconds
Previous window count: 100
Current window count: 30

Estimated = 30 + 100 × (60 − 15) / 60
          = 30 + 75
          = 105
```

**Advantages:** smoother than fixed windows with far less memory than a request log.

**Problem:** it is an approximation, not an exact count.

---

### 4. Token Bucket (Recommended for APIs)

Each client has a bucket with a maximum number of tokens. Tokens are added at a fixed rate; each request consumes one token.

```text
Bucket capacity: 100 tokens
Refill rate: 10 tokens/second

Request → token available? → consume token → allow
                         └→ no token       → reject
```

**Advantages:** allows controlled short bursts while maintaining an average rate; commonly used for APIs.

**Problem:** refill is a read-modify-write operation. A request reads the last-update timestamp and token count, calculates tokens to add from elapsed time, then writes the new count and timestamp. If two requests read the same stale state, both can consume a token that should not exist.

Use a Redis Lua script so no other command interleaves during the refill-and-consume operation:

```text
read last_update and token_count
elapsed = now - last_update
token_count = min(capacity, token_count + elapsed × refill_rate)

if token_count >= 1
  token_count = token_count - 1
  allowed = true
else
  allowed = false
end

write token_count and now
return allowed
```

---

### 5. Leaky Bucket

Requests enter a queue and leave at a constant rate.

```text
Incoming requests → queue → process at 10 requests/second
```

**Advantages:** produces smooth outgoing traffic and protects a slow downstream system.

**Problem:** queued requests add latency; a full queue rejects requests.

---

## 🔹 7. Distributed Rate Limiting

With multiple API servers, local in-memory counters are incorrect because the same client can hit different servers.

```text
Client → API Server 1 → local count = 60
Client → API Server 2 → local count = 60

Actual total = 120 requests
```

Use a shared store such as Redis. The check and update must be atomic to avoid race conditions. Redis `INCR` works for a fixed window; a Lua script is often used when several reads and writes must occur together, such as token-bucket refill and consume.

Production systems commonly use two tiers. Each API server keeps a local in-process token bucket, avoiding a network call on the hot path, and periodically syncs quota from central Redis. Set local limits slightly below the global limit to avoid over-granting. This reduces Redis load and directly helps with hot keys.

Sliding windows and token-bucket refill depend on timestamps. If server clocks drift, a client hitting different servers can receive inconsistent limits. Use a monotonic clock for local refill calculations; for shared Redis state, rely on Redis's single clock so every server uses the same time source.

---

## 🔹 8. Failure Handling

### Redis Unavailable

Pick a policy based on the endpoint:

* **Fail open** — allow the request. Better availability, weaker abuse protection.
* **Fail closed** — reject the request. Better protection, but can block legitimate traffic.

For example, fail closed for login or password-reset endpoints, and fail open for ordinary read APIs if availability is more important.

### Hot Keys

A popular shared API key can overload one Redis key. Use local limits, key sharding where safe, or a dedicated limiter tier for extreme traffic.

---

## 🔹 9. Scaling Strategy

* Put coarse limits at CDN/load-balancer level to drop obvious abuse early.
* Apply precise user or API-key limits at the API gateway.
* Use Redis Cluster and shard rate-limit keys by hash slot so load is distributed evenly across nodes.
* Keep rate-limit rules in configuration so limits can change without deployment.
* Monitor allowed, blocked, and Redis-error rates by endpoint and client type.

---

## 🔹 10. Common Interview Questions

### Q. Which algorithm would you use for a public API?

**Answer:** Token bucket is a strong default because it permits small bursts but enforces a stable average rate. Sliding window counter is useful when smoother enforcement is needed with limited memory.

---

### Q. Why use Redis?

**Answer:** It is fast, supports expiry, and provides atomic operations shared across all API servers.

---

### Q. What HTTP status code is returned when the limit is exceeded?

**Answer:** `429 Too Many Requests`, usually with a `Retry-After` header.

---

### Q. Where should the limiter run?

**Answer:** Prefer the API gateway or reverse proxy so rejected requests do not consume application resources. Application-level limits can add rules that require user or business context.

---

## 🎯 One-Line Answer

> A rate limiter protects services by tracking requests per client over time, usually in a shared store such as Redis, and returning HTTP 429 when the chosen limit is exceeded. Token bucket is a common API choice because it supports controlled bursts.
