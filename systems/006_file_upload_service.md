# File Upload Service — System Design

## Functional Requirements (what it does)

1. **Upload a file** — authenticated user sends a file, gets back an identifier.
2. **Retrieve / download a file** — by id, with authorization.
3. **Delete a file.**
4. **List a user's files** (metadata, pagination).
5. **Optionally share** — generate a public link.
6. **Post-process** — thumbnails, virus scan, transcoding (if it's an image/video service).

> Nothing here says *how* to store — storage is a non-functional concern.

## Non-Functional Requirements (how well)

1. **Durability** — files must never be lost.
2. **Scalability** — many concurrent uploads, growing to petabytes, without redesign.
3. **Availability** — uploads/downloads keep working even if a component is degraded.
4. **Low latency for downloads** — especially repeat/hot content.
5. **Security** — private by default, per-user authorization, malware-free.
6. **Cost efficiency** — storage and bandwidth dominate cost at scale.
7. **Consistency** — a file is fully available or not visible at all; no half-uploads.

## Requirement → Decision → Why

### Durability → S3 / object storage (not local disk, not DB blobs)
Local disk dies with the container and isn't shared across Puma instances; Postgres blobs bloat WAL, backups, and replication. S3 gives 11 nines durability, versioning, and cross-region replication for free. The requirement makes object storage non-negotiable.

### Scalability → client uploads directly via pre-signed URL; app servers handle only metadata
If bytes flowed through Rails, one 2 GB upload would occupy a Puma thread and consume memory, capping throughput at `threads × bandwidth`. Routing bytes *around* the app tier makes the web tier horizontally trivial — it only does cheap JSON + DB work.

### Separate data types → metadata in Postgres, bytes in S3, joined by `s3_key`
Bytes go to S3, but you still need to find and authorize files — that's relational data. The two stores have different scaling and access patterns, so splitting them lets each scale independently.

Metadata table:

    id, owner_id, s3_key, filename, size, content_type, checksum, status, created_at

### Low latency + cost → pre-signed GET URLs, CDN (CloudFront) for hot/public files
S3 alone means every download hits S3 and pays egress per request. A CDN caches at the edge, cutting latency and egress cost.

### Security → private bucket, no public ACLs, short-lived signed URLs, auth before issuing
The signed URL *is* the authorization token — time-limited and scoped to one object. The bucket stays private so nobody can enumerate.

### Malware-free + validated → async scan in a Sidekiq worker, gated by `status`
You can't scan inline (latency) and can't trust the client's content-type. File lands as `status = pending`, a worker validates/scans, then flips to `ready`. This forces the **status state machine**.

### Consistency / no orphans → status field + sweeper job
A pre-signed URL can be issued and never used, leaving a `pending` row and a stray S3 object. Reads filter on `status = ready`; a periodic job reaps stale rows and unreferenced objects.

### Cost efficiency → dedup by content hash (SHA-256) + lifecycle to cold storage
Hash the content; identical files reuse the same `s3_key` (mind reference counting for deletes). Old files move to Glacier via lifecycle rules. Justifies the `checksum` column.

## Core Flow

1. Client calls `POST /uploads` with filename, size, content-type.
2. Rails validates (auth, size, type), inserts metadata row `status = pending`, returns a **pre-signed PUT URL** for a generated `s3_key`.
3. Client uploads bytes **directly to S3** — app never touches the stream.
4. S3 event → SQS → Sidekiq worker verifies object, flips `status = ready`, queues virus scan / thumbnails.

## Interview Follow-ups to Be Ready For

- **Large files / resumability** — S3 multipart upload; client retries failed parts, then completes.
- **Downloads** — short-expiry pre-signed GET URLs; CloudFront for public/hot content.
- **Security** — private bucket, per-user auth, content-type + size validation, sandboxed malware scan, rate limiting.
- **Dedup** — SHA-256 checksum; reuse `s3_key`; reference-count for deletes.
- **Orphans** — sweeper job deletes stale `pending` rows and unreferenced S3 objects.
- **Consistency** — S3 is source of truth for bytes, Postgres for metadata; a file is "real" only at `status = ready`.
- **Trade-off** — the presign → upload → confirm flow is eventually consistent; use the SQS event path rather than trusting the client callback.

## The Thread to Pull

Don't say "I'll use S3" first — it sounds arbitrary. Say it *because* durability + scale + cost leave no other option. **Requirement → constraint → component** is what separates a senior answer from a memorized one.

The two requirements doing the most work: **scalability** (forces pre-signed direct upload, bytes bypass the app tier) and **security/consistency** (forces the pending → ready state machine).
