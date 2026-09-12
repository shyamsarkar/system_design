# Pastebin — System Design

## 1. Requirements

### Functional

- User pastes text and gets a unique, shareable URL.
- User retrieves paste content by URL.
- Paste has a retention policy (10M, 1H, 1D, 1W, 1M, 6M, 1Y, or Indefinite).
- **Terminology:** “Never expire” is the user-facing term for indefinite retention; internally, it is represented as `expires_at = NULL`.
- Visibility: Public (listed + indexed), Unlisted (accessible by URL, not indexed), Private (owner-only).
- Optional syntax highlighting.
- Anonymous (guest) creation allowed; account-based creation for management features.
- Account users can edit/delete their own pastes; guests cannot delete/edit.

### Non-functional

- Read-heavy workload (read:write ≈ 10:1 or higher).
- Low read latency (paste links are shared in chat/forums — users expect instant load).
- High read availability (a dead link breaks the conversation).
- Durability: once written, a paste must not silently disappear (unless expired).
- Storage cost must stay bounded despite the "Never expire" option.
- Paste content is immutable after creation (no in-place edits for guests; rare edits for accounts).

---

## 2. Capacity Estimation

| Metric | Assumption |
|---|---|
| Pastes created/day | 10M |
| Average paste size | ~4 KB (raw text/code) |
| Read:write ratio | 10:1 |
| Never-expire fraction | ~90% |
| Compression ratio | 3–5x (text compresses well) |
| Dedup savings | ~20–30% (logs, boilerplate repeat) |

**Writes:** 10M × 4 KB = ~40 GB/day raw → ~10–13 GB/day after compression + dedup.

**Reads:** 100M reads/day = ~1,200 reads/sec average, ~5,000/sec peak.

**Storage growth:** ~4–5 TB/year compressed (net of dedup), plus PostgreSQL metadata/index growth. Indefinite content makes logical storage monotonically grow; the cost-control strategy therefore relies on compression, deduplication, rate limiting, and lifecycle-based archival rather than silently deleting retained content.

**Bandwidth:** 100M reads × 4 KB = ~400 GB/day outbound.

**Key insight:** Pastebin is storage-bound, not CPU/network-bound. The design must control unbounded storage growth from never-expire pastes.

---

## 3. High-Level Architecture

```
                    ┌─────────────┐
                    │   Client    │  (web UI / API consumer)
                    └──────┬──────┘
                           │
                    ┌──────▼───────┐
                    │ Load Balancer│
                    └──────┬───────┘
                           │
              ┌────────────▼─────────────┐
              │   Application Layer      │
              │  (stateless web servers) │
              │  - create/read requests  │
              │  - auth, expiry logic    │
              │  - rate limiting         │
              └─────┬─────────────┬──────┘
                    │             │
         ┌──────────▼────┐   ┌────▼───────────┐
         │ Metadata DB   │   │ Object Storage │
         │ (PostgreSQL)  │   │  (S3)          │
         │ - paste_id PK │   │ - key=hash     │
         │ - user/expiry │   │ - compressed   │
         └───────────────┘   └────────────────┘
                    │
              ┌─────▼───────┐
              │   Redis     │  (metadata + rendered HTML cache)
              └─────────────┘
```

**Why split metadata from blob:**

- Metadata is small, queryable, needs indexing (by paste_id, user_id, expires_at).
- Blob content is large, immutable, accessed by key only — ideal for object storage.
- Mixing them wastes DB resources and makes backups expensive.

---

## 4. Data Model

### Paste metadata (PostgreSQL)

```sql
CREATE TABLE pastes (
    paste_id        VARCHAR(8) PRIMARY KEY,      -- URL key, e.g. 'aB3xK9qf'
    user_id         BIGINT,                       -- NULL for guest
    content_hash    CHAR(64) NOT NULL,            -- SHA-256, for dedup
    expires_at      TIMESTAMP NULL,               -- NULL = never expire
    created_at      TIMESTAMP NOT NULL DEFAULT NOW(),
    last_viewed_at  TIMESTAMP NOT NULL DEFAULT NOW(),
    view_count      BIGINT NOT NULL DEFAULT 0,
    visibility      SMALLINT NOT NULL,             -- 0=public, 1=unlisted, 2=private
    size_bytes      INT NOT NULL,
    syntax_format   VARCHAR(20),
    deleted         BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE INDEX idx_pastes_user_id        ON pastes(user_id) WHERE user_id IS NOT NULL;
CREATE INDEX idx_pastes_expires_at     ON pastes(expires_at) WHERE expires_at IS NOT NULL;
CREATE INDEX idx_pastes_last_viewed_at ON pastes(last_viewed_at);
CREATE INDEX idx_pastes_content_hash   ON pastes(content_hash);
```

### Blob storage (S3)

```
bucket: pastebin-blobs
key:    <content_hash>        # e.g. sha256 hex
value:  gzip-compressed paste content
```

### Content hash refcount table

```sql
CREATE TABLE blob_refcounts (
    content_hash CHAR(64) PRIMARY KEY,
    refcount     BIGINT NOT NULL,
    size_bytes   INT NOT NULL
);
```

### Storage consistency

PostgreSQL and S3 do not share one ACID transaction, so create/delete operations must be designed for partial failure.

**Create:**

1. Compute the content hash and deterministically choose the S3 key.
2. Upload the blob using an idempotent operation.
3. Insert the metadata/refcount records transactionally in PostgreSQL.
4. If the DB transaction fails after the upload, the object becomes an orphan and is removed later by an orphan-blob GC job.
5. If the upload fails, do not create a visible paste metadata row.

**Delete:**

1. Mark/delete the paste metadata and decrement the DB refcount in one transaction.
2. Do not require an immediate S3 delete.
3. A background job deletes objects whose authoritative DB refcount is zero and which are past a safety delay.

This deliberately favors **eventual cleanup** over trying to make PostgreSQL and S3 behave like one distributed transaction.

**Why key blobs by content hash:**

- Identical pastes (error logs, boilerplate, repeated snippets) share one blob.
- Deleting a paste decrements the refcount; the blob is purged only when refcount = 0.
- Typical dedup savings: 20–30% of raw paste volume.

---

## 5. URL Scheme — the Paste Key

- 8-character base62 (`a-zA-Z0-9`) → 62⁸ ≈ 218 trillion combinations.
- Generated randomly, with a DB uniqueness check (retry on rare collision).
- **Avoid sequential IDs** — enumerable, so an attacker can scrape every paste.
- Short enough to share in chat; long enough to be practically unguessable.

---

## 6. Core Flows

### Create paste

1. Client POSTs content + options (expiry, visibility, syntax).
2. App server validates size (cap: 512 KB free / 10 MB paid).
3. Rate-limit check (per IP for guests, per user_id for accounts).
4. Compute `content_hash = SHA-256(content)`.
5. Compress content (gzip/zstd).
6. Dedup using `content_hash` with a unique constraint. Concurrent creators race safely on the same hash; the database has one authoritative refcount row. Uploads are idempotent because the S3 key is deterministic (`content_hash`).
7. Generate random 8-char `paste_id`.
8. Insert metadata row into `pastes`.
9. Return URL: `pastebin.com/<paste_id>`.

### Read paste

1. Client GETs `/<paste_id>`.
2. Public immutable content is served through the CDN when possible.
3. On CDN miss, check Redis for metadata/rendered content.
4. On cache miss, query PostgreSQL for metadata by `paste_id`.
5. Expiry check: if `expires_at < NOW()`, return **410 Gone** and queue cleanup.
6. Fetch blob from S3 by `content_hash`, decompress.
7. Apply syntax highlighting, render HTML.
8. Update `view_count` asynchronously/batched; `last_viewed_at` is an application-observed activity signal, not an exact global read counter because CDN hits may bypass the application.
9. Cache the rendered result in Redis and the response at the CDN with a TTL consistent with the retention policy.
10. Return content.

---

## 7. Expiry and Garbage Collection

This is the most interesting design problem.

### Two retention types

1. **Timed expiry** (`expires_at` is set) — paste expires at a known time.
2. **Indefinite retention** (`expires_at IS NULL`) — no automatic expiry is assigned. Cold content may be moved to cheaper storage, but it is not deleted merely because it is old.

### Strategy: lazy expiry + background cleanup

**Lazy expiry (on read):**

- When a timed paste is read, check `expires_at`.
- If expired → return 410, mark `deleted = TRUE`, queue cleanup.
- This avoids requiring a hot-path scan of all expired rows.

**Background cleanup (timed pastes):**

- A periodic job scans/indexes `expires_at` to clean expired rows that nobody ever reads again.
- Process rows in batches using `FOR UPDATE SKIP LOCKED` or an equivalent job-claiming mechanism.
- Decrement `blob_refcounts` for deleted pastes; blob deletion remains asynchronous.

**Indefinite-retention content:**

- Do not delete solely because `last_viewed_at` is old.
- Move cold objects from S3 Standard to cheaper storage tiers using lifecycle policies where appropriate.
- If product requirements later introduce a storage-retention policy, it should be an explicit product rule rather than being hidden inside the garbage collector.

### Blob cleanup

- Deleting a paste does **not** delete the blob immediately.
- Decrement refcount for the paste's `content_hash`.
- A separate job purges S3 objects where `refcount = 0`.
- Avoids deleting a blob still referenced by another paste.

### Why this works

- Timed-expiry pastes are cleaned lazily — zero background cost until someone reads them.
- Indefinite-retention pastes are not deleted based on inactivity; cold content may instead be moved to cheaper storage tiers.

---

## 8. Caching Strategy

Reads dominate, and traffic is highly skewed (recently-shared pastes are hot).

### CDN cache (public pastes)

- Cache rendered paste HTML at the CDN edge.
- Key = `paste_id`.
- Content is immutable, so long cache lifetimes are safe from content edits.
- Timed-expiry responses must respect the remaining retention window; purge/invalidate on explicit deletion when supported.
- CDN access logs/analytics can be used as an activity signal because CDN hits may bypass the application.

### Redis cache (metadata + rendered)

- `paste_id → metadata` (avoids PostgreSQL hit on every read).
- `paste_id → rendered HTML` (avoids re-rendering syntax highlighting).
- TTL = remaining time to expiry, or 1 hour for never-expire (then refresh).

### Expected hit rate

- Traffic is expected to be highly skewed toward recently shared pastes.
- Even a modest cache should absorb a large fraction of reads, reducing PostgreSQL and S3 pressure.
- The exact hit rate should be validated with production metrics rather than assuming a fixed 80/20 split.

### Invalidation

- Pastes are immutable — no stale cache problem on content.
- Only triggers: expiry (lazy, checked on read) and explicit deletion (cache purge).

---

## 9. Storage Cost Control

The combination of never-expire + anonymous + no-delete creates unbounded storage growth. No single lever solves it; the design uses all of:

| Lever | Effect |
|---|---|
| Per-paste size cap (512 KB free, 10 MB paid) | Bounds worst case per paste |
| Compression (gzip/zstd) | 3–5x reduction on text |
| Content-hash dedup | 20–30% savings (identical pastes stored once) |
| Creation rate limiting (per IP, per token) | Prevents abuse-as-storage |
| Storage tiering / archival | Moves cold indefinite-retention content from S3 Standard to cheaper storage tiers |

---

## 10. Security and Abuse

- **Rate limiting on creation** — per IP (guests) and per user_id (accounts). Anonymous pastes are the primary abuse vector (spam, illegal content, storage abuse).
- **Content moderation** — automated spam/abuse detection: keyword filters, duplicate detection, URL scanning.
- **Report/DMCA system** — manual takedown path with 24-hour SLA.
- **No guest delete/edit** — the server cannot distinguish the original creator from any other visitor, so guest pastes are create-only. Prevents vandalism.
- **Private paste access control** — private pastes require auth; unlisted pastes are unguessable but accessible by URL.
- **Avoid sequential IDs** — prevents enumeration/scraping attacks.

---

## 11. Scalability and Bottlenecks

| Component | Bottleneck | Mitigation |
|---|---|---|
| Metadata DB | PostgreSQL metadata and indexes grow with paste count | Start with one primary + indexes; add read replicas, partitioning/archival, and only consider sharding when measurable limits are reached |
| Object storage | S3 handles natively | No sharding needed; lifecycle policies for tiered storage |
| App servers | Stateless, easy to scale | Horizontal scaling behind LB |
| Garbage collection | Sweeper scan cost at scale | Partition table by `created_at` or `expires_at`; batch processing |
| Cache invalidation | Expiry storms if many pastes expire simultaneously | Stagger expiry times; use TTL-based eviction as fallback |

---

## 12. Monitoring and Observability

- Paste creation rate, read rate, cache hit ratio.
- Storage growth rate (track indefinite-retention accumulation — the key business metric).
- PostgreSQL table/index size and query latency as metadata volume grows.
- Expiry/cleanup job throughput (are we keeping up with deletions?).
- 410/404 rate (pastes expiring or not found).
- Abuse/moderation queue depth.
- S3 storage cost per month (alert if growth exceeds forecast).

---

## 13. API Design (REST)

### Create paste

```
POST /api/paste
Content-Type: application/json

{
  "content": "def hello; puts 'hi'; end",
  "expire": "1D",           // 10M, 1H, 1D, 1W, 1M, 6M, 1Y, INDEFINITE
  "visibility": "unlisted", // public, unlisted, private
  "syntax": "ruby",
  "name": "My snippet"
}

→ 200 OK
{
  "url": "https://pastebin.com/aB3xK9qf",
  "paste_id": "aB3xK9qf",
  "expires_at": "2025-09-06T12:00:00Z"
}
```

### Read paste

```
GET /api/paste/<paste_id>

→ 200 OK
{
  "content": "def hello; puts 'hi'; end",
  "syntax": "ruby",
  "created_at": "2025-09-05T12:00:00Z",
  "expires_at": "2025-09-06T12:00:00Z"
}

→ 410 Gone (if expired)
→ 404 Not Found (if deleted/never existed)
```

### Delete paste (account only)

```
DELETE /api/paste/<paste_id>
Authorization: Bearer <user_token>

→ 200 OK
→ 401 Unauthorized (guest)
→ 403 Forbidden (not owner)
```

---

## 14. Summary

Pastebin is deceptively simple. The core design challenge is not serving reads or generating URLs — it is **keeping storage and metadata growth sustainable while supporting indefinite retention, anonymous creation, and high read traffic.**

The solution is a combination of:

1. **Compression + dedup** to shrink what is stored.
2. **Size caps + rate limiting** to prevent abuse-as-storage.
3. **Lazy expiry** for timed pastes (zero background cost).
4. **Background cleanup + asynchronous blob GC** for timed expirations and partial-failure recovery.
5. **CDN + Redis caching** to handle the read-heavy workload cheaply.
6. **Metadata/blob split** to keep the queryable layer small and the bulk storage cheap.
7. **Storage tiering** to make indefinite retention economically sustainable.

The important trade-off is explicit: the system preserves the logical retention guarantee while using compression, deduplication, lifecycle policies, and asynchronous cleanup to control infrastructure cost.
