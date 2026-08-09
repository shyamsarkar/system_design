# 🔗 URL Shortener Design (Interview Ready)

## 🔹 1. Requirements

### Functional Requirements

* Convert long URL → short URL
* Redirect short URL → original URL
* Handle high traffic (millions of requests)

### Non-Functional Requirements

* High availability
* Low latency ⚡
* High read throughput

---

## 🔹 2. High-Level Flow

```text
User → API → Generate Short URL → Store in DB
User → Short URL → Redirect Service → Original URL
```

---

## 🔹 3. Core Components

### 1. API Server

* Accepts long URL
* Generates short URL
* Stores mapping

### 2. Database

Stores mapping:

```text
short_code → long_url
```

### 3. Redirect Service

* Reads short URL
* Fetches original URL
* Redirects user

---

## 🔹 4. Short URL Generation (🔥 Most Important)

### Approach 1: Auto Increment ID + Base62 (Recommended)

```text
ID = 1001 → Base62 → "g7"
```

**Base62 characters:**

* a-z (26)
* A-Z (26)
* 0-9 (10)

👉 Total = 62 characters

**Advantages:**

* Unique
* Short
* No collision

---

### Approach 2: Hashing (MD5/SHA)

```text
hash(long_url) → first 6 chars
```

**Problem:**

* Collision risk (same short URL for different inputs)

---

## 🔹 5. Database Design

```sql
urls
-----
id (auto increment)
short_code (unique)
long_url
created_at
```

---

## 🔹 6. Read Optimization (Very Important)

* System is **read-heavy**

### Use Cache (Redis)

```text
Short URL → Cache → DB (if miss)
```

**Benefits:**

* Faster response
* Reduced DB load

---

## 🔹 7. Scaling Strategy

### Horizontal Scaling

* Multiple API servers

### Load Balancer

* Distribute incoming traffic

### Database Scaling

* Read replicas
* Sharding (for very large scale)

---

## 🔹 8. Handling Collisions

* Check DB before inserting
* Regenerate short code if exists (if using hashing)

---

## 🔹 9. Expiry (Optional)

```text
short_url expires after X days
```

* Clean up old records

---

## 🔹 10. Analytics (Optional Feature)

Track:

* Click count
* Location
* Device

---

## 🔹 11. Final Architecture

```text
Client
  ↓
Load Balancer
  ↓
API Servers
  ↓
Cache (Redis)
  ↓
Database
```

---

## 🔥 Advanced Considerations

* Hot URLs (cache frequently accessed links)
* Rate limiting (prevent abuse)
* Custom short URLs (user-defined aliases)

---

## 🎯 One-Line Answer

> A URL shortener generates unique short codes using Base62 encoding of IDs, stores mappings in a database, and uses caching like Redis to efficiently handle high read traffic.
