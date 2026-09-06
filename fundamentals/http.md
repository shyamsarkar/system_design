# 🌐 HTTP & HTTPS Fundamentals (Interview Ready)

## 🔹 1. Overview: What Problem Does HTTP Solve?

**HTTP (Hypertext Transfer Protocol)** is an application-layer (OSI Layer 7) protocol designed for transmitting hypermedia documents. It operates on a **client-server**, **stateless**, **request-response** architecture.

### Why HTTP Matters in System Design
* **Universal API Contract:** It is the foundational communication standard between clients (web, mobile, IoT) and backend services, as well as between microservices.
* **Latency & Performance:** Understanding connection reuse, multiplexing, and protocol overhead directly impacts end-user latency.
* **Caching & Cost Optimization:** Proper HTTP cache headers reduce origin database load and bandwidth costs via CDNs.
* **Reliability & Idempotency:** Correct use of HTTP verbs and status codes guarantees safe retries across distributed networks.

---

## 🔹 2. Anatomy of HTTP Request & Response

Every HTTP interaction consists of a plaintext (in HTTP/1.x) or binary framed (in HTTP/2 & HTTP/3) request and response.

### 1. HTTP Request Structure

```text
+-------------------------------------------------------+
| Method | Request-URI | HTTP-Version                   |  <- Request Line
+-------------------------------------------------------+
| Header-Name: Header-Value                             |
| Header-Name: Header-Value                             |  <- Headers
+-------------------------------------------------------+
| <CRLF> (Empty Line)                                   |
+-------------------------------------------------------+
| Message Body (JSON, XML, binary payload, etc.)        |  <- Body (Optional)
+-------------------------------------------------------+
```

#### Raw HTTP Request Example:
```http
POST /api/v1/orders HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0
Content-Type: application/json
Content-Length: 48
Authorization: Bearer eyJhbGciOi...
Idempotency-Key: 7b8b4d24-39f5-4672-91aa

{"item_id": 1042, "quantity": 2, "price": 49.99}
```

---

### 2. HTTP Response Structure

```text
+-------------------------------------------------------+
| HTTP-Version | Status-Code | Reason-Phrase            |  <- Status Line
+-------------------------------------------------------+
| Header-Name: Header-Value                             |
| Header-Name: Header-Value                             |  <- Headers
+-------------------------------------------------------+
| <CRLF> (Empty Line)                                   |
+-------------------------------------------------------+
| Response Body (HTML, JSON, image bytes, etc.)         |  <- Body (Optional)
+-------------------------------------------------------+
```

#### Raw HTTP Response Example:
```http
HTTP/1.1 201 Created
Date: Fri, 04 Sep 2026 08:30:00 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 62
Location: /api/v1/orders/89412
Cache-Control: no-cache

{"order_id": 89412, "status": "PENDING", "created_at": 1788510600}
```

---

## 🔹 3. HTTP Methods (Verbs) & Semantics

In RESTful system design, methods indicate the action to be performed on a resource. Two critical properties define each method:
1. **Safe:** The method does not alter server state (read-only).
2. **Idempotent:** Making multiple identical requests produces the exact same server side-effects as making a single request (`f(f(x)) = f(x)`).

| Method | Safe? | Idempotent? | Cacheable? | Typical Usage |
| :--- | :---: | :---: | :---: | :--- |
| **GET** | ✅ Yes | ✅ Yes | ✅ Yes | Retrieve resource representations. Must never modify state. |
| **POST** | ❌ No | ❌ No | ⚠️ Conditionally | Create new subordinate resource, trigger processing, or submit forms. |
| **PUT** | ❌ No | ✅ Yes | ❌ No | Replace the entire target resource with the uploaded payload. |
| **PATCH** | ❌ No | ❌ No* | ❌ No | Apply partial modifications to a resource (*can be idempotent if designed so). |
| **DELETE** | ❌ No | ✅ Yes | ❌ No | Remove the specified resource. |
| **HEAD** | ✅ Yes | ✅ Yes | ✅ Yes | Identical to `GET`, but server transfers headers only (no body). Used to check file size/caching. |
| **OPTIONS**| ✅ Yes | ✅ Yes | ❌ No | Query supported methods (used heavily in CORS preflight checks). |

> 🔥 **System Design Impact — Idempotency & Retries:**  
> When network timeouts occur, the client cannot know if the request reached the server.  
> - **GET, PUT, DELETE** can be safely retried automatically.  
> - **POST** cannot be retried blindly without causing duplicate operations (e.g., charging a card twice). It requires an **Idempotency Key**.

---

## 🔹 4. HTTP Status Codes (Interview Essentials)

Status codes indicate the outcome of the client's request.

```text
1xx: Informational   (Request received, continuing process)
2xx: Success         (Action successfully received, understood, and accepted)
3xx: Redirection     (Further action must be taken to complete request)
4xx: Client Error    (Request contains bad syntax or cannot be fulfilled)
5xx: Server Error    (Server failed to fulfill an apparently valid request)
```

### 1xx — Informational
* **100 Continue:** Server received initial request headers; client should proceed to send the body.
* **101 Switching Protocols:** Client requested protocol change; server agreed (e.g., upgrading HTTP/1.1 to WebSocket).

### 2xx — Success
* **200 OK:** Standard success response for `GET`, `PUT`, or `POST`.
* **201 Created:** Resource successfully created (standard for `POST`; should include `Location` header).
* **202 Accepted:** Request accepted for processing, but processing is not complete (used in asynchronous/event-driven architectures).
* **204 No Content:** Request succeeded, but there is no payload to return (common for `DELETE` or `PUT`).
* **206 Partial Content:** Server fulfilled partial `GET` request (used in video streaming, download resuming via `Range` header).

### 3xx — Redirection
* **301 Moved Permanently:** Target resource assigned a new permanent URI. Browsers and search engines aggressively cache this redirect.
* **302 Found (Temporary Redirect):** Resource temporarily resides under a different URI. Browsers do not cache this permanently.
* **304 Not Modified:** Client's cached copy is still valid (conditional request using `If-None-Match` or `If-Modified-Since`). Saves bandwidth.
* **307 Temporary Redirect:** Like 302, but guarantees the HTTP method cannot be changed (e.g., POST remains POST).
* **308 Permanent Redirect:** Like 301, but guarantees the HTTP method cannot be changed.

### 4xx — Client Errors
* **400 Bad Request:** Malformed syntax, invalid JSON, or failed input validation.
* **401 Unauthorized:** Actually means **Unauthenticated**. Missing or invalid credentials.
* **403 Forbidden:** Client is authenticated, but lacks permissions (authorization failure).
* **404 Not Found:** Requested resource does not exist.
* **405 Method Not Allowed:** Endpoint exists, but does not support the given HTTP verb (e.g., `POST` on a read-only endpoint).
* **409 Conflict:** State conflict on the server (e.g., unique key violation, optimistic concurrency lock failure).
* **429 Too Many Requests:** Client exceeded rate limits (should return with `Retry-After` header).

### 5xx — Server Errors
* **500 Internal Server Error:** Generic unhandled error or server crash.
* **502 Bad Gateway:** The proxy/load balancer received an invalid or corrupt response from the upstream backend server.
* **503 Service Unavailable:** Server overloaded or down for maintenance (circuit breaker tripped; upstream capacity exhausted).
* **504 Gateway Timeout:** The proxy/load balancer did not receive a timely response from upstream (upstream service or database is slow/deadlocked).

---

## 🔹 5. Core HTTP Headers

Headers pass metadata between client and server.

```text
Request / Response Headers
├── Identification & Context (Host, User-Agent, Referer)
├── Content Negotiation (Content-Type, Content-Length, Accept, Accept-Encoding)
├── Caching & Freshness (Cache-Control, ETag, If-None-Match, Last-Modified)
├── Security & Auth (Authorization, Strict-Transport-Security, Content-Security-Policy)
├── Cross-Origin (Access-Control-Allow-Origin, Access-Control-Allow-Methods)
└── Observability & Proxies (X-Forwarded-For, X-Forwarded-Proto, X-Request-ID)
```

### 1. Caching Headers (Critical for Scale)
* **`Cache-Control`:** Directives for caching mechanisms:
  * `max-age=3600`: Cacheable for 1 hour.
  * `s-maxage=86400`: Overrides `max-age` for shared caches / CDNs.
  * `no-cache`: Must validate with the origin server before using cached copy.
  * `no-store`: Never cache any part of request/response (sensitive data).
  * `immutable`: Content will never change (ideal for versioned static assets: `bundle.a8f91b.js`).
* **`ETag` (Entity Tag):** A unique hash/fingerprint of the resource version.
  * Client sends `If-None-Match: "hash123"` in subsequent requests.
  * If unchanged, server returns `304 Not Modified` with zero body transfer.

### 2. Security Headers
* **`Strict-Transport-Security` (HSTS):** Enforces HTTPS connections only, preventing SSL stripping attacks.
* **`Content-Security-Policy` (CSP):** Restricts sources from which scripts, styles, and images can load (mitigates XSS).
* **`X-Content-Type-Options: nosniff`:** Prevents MIME-type sniffing.

### 3. Cross-Origin Resource Sharing (CORS)
Browsers enforce the **Same-Origin Policy (SOP)**. When cross-origin requests occur:
1. Browser sends a preflight `OPTIONS` request.
2. Server responds with allowed origins, headers, and methods:
   * `Access-Control-Allow-Origin: https://app.example.com`
   * `Access-Control-Allow-Methods: GET, POST, PUT, DELETE`
   * `Access-Control-Allow-Headers: Content-Type, Authorization`

### 4. Distributed Tracing & Proxies
* **`X-Forwarded-For`:** Client IP address appended by load balancers and reverse proxies.
* **`X-Forwarded-Proto`:** Protocol used by original client (`http` vs `https`).
* **`X-Request-ID` / `Traceparent`:** Distributed tracing correlation ID passed across microservices.

---

## 🔹 6. State Management & Cookies

Because HTTP is inherently stateless, state is maintained at the application layer via **Cookies** or **Bearer Tokens**.

```text
Client                                             Server
  │                                                   │
  │  POST /login (Credentials)                        │
  │──────────────────────────────────────────────────>│ Authenticate
  │                                                   │ Generate Session
  │  200 OK                                           │
  │  Set-Cookie: session_id=xyz; HttpOnly; Secure     │
  │<──────────────────────────────────────────────────│
  │                                                   │
  │  GET /dashboard                                   │
  │  Cookie: session_id=xyz                           │
  │──────────────────────────────────────────────────>│ Validate Session
```

### Essential Cookie Security Attributes

| Attribute | Purpose | Security Protection |
| :--- | :--- | :--- |
| **`HttpOnly`** | Prevents client-side scripts (`document.cookie`) from accessing the cookie. | Mitigates **Cross-Site Scripting (XSS)** attacks. |
| **`Secure`** | Cookie only sent over encrypted HTTPS connections. | Prevents **Man-In-The-Middle (MITM)** packet sniffing. |
| **`SameSite=Strict`** | Cookie only sent in first-party context (same site). | Mitigates **Cross-Site Request Forgery (CSRF)**. |
| **`SameSite=Lax`** | Cookie sent when user navigates to origin site from external link. | Balanced default for user experience & CSRF protection. |
| **`Domain & Path`**| Defines the scope and URL path where the cookie is valid. | Prevents cookie leaking to unauthorized subdomains. |

---

## 🔹 7. Connection Management & Keep-Alive

### The Problem in Early HTTP (HTTP/1.0)
In HTTP/1.0, every single request required establishing a brand new TCP connection:
```text
TCP 3-Way Handshake → Request → Response → TCP Teardown
```
For a web page requesting 50 assets (CSS, JS, images), this resulted in 50 consecutive TCP handshakes, causing massive network latency and server connection bloat.

### HTTP/1.1 Persistent Connections (`Keep-Alive`)
HTTP/1.1 introduced persistent connections by default:
* **`Connection: keep-alive`** keeps the TCP socket open for subsequent requests.
* **`Keep-Alive: timeout=5, max=100`** instructs client on connection reuse bounds.

```text
Client                                                Server
  │                                                      │
  │─── TCP 3-Way Handshake ─────────────────────────────>│ (Established once)
  │─── Request 1 (index.html) ──────────────────────────>│
  │<── Response 1 ───────────────────────────────────────│
  │─── Request 2 (style.css) ───────────────────────────>│ (Reuses socket)
  │<── Response 2 ───────────────────────────────────────│
  │─── Request 3 (script.js) ───────────────────────────>│ (Reuses socket)
  │<── Response 3 ───────────────────────────────────────│
```

### System Design Benefit: Connection Pooling
* Backend services (e.g., API Gateway to microservices, or microservice to database) maintain **connection pools**.
* Pre-warmed sockets eliminate handshake latency and prevent `TIME_WAIT` socket exhaustion on high-throughput servers.

---

## 🔹 8. Protocol Evolution: HTTP/1.1 vs HTTP/2 vs HTTP/3

```text
HTTP/1.1 (1997)             HTTP/2 (2015)                HTTP/3 (2020+)
┌───────────────┐           ┌───────────────────┐        ┌───────────────────┐
│ Text Protocol │           │ Binary Framing    │        │ Binary Framing    │
├───────────────┤           ├───────────────────┤        ├───────────────────┤
│ TCP (1 req/c) │           │ Multiplexing over │        │ Multiplexing over │
│ HoL Blocking  │           │ single TCP stream │        │ QUIC (UDP)        │
└───────────────┘           └───────────────────┘        └───────────────────┘
```

### 1. HTTP/1.1 — The Text Standard
* **Mechanism:** Plaintext headers and bodies.
* **Bottleneck — Application-Level Head-of-Line (HoL) Blocking:**  
  On a single TCP connection, requests must be processed serially. If Request 1 takes 2 seconds, Requests 2 and 3 must wait in line.
* **Workarounds:**
  * **Domain Sharding:** Browsers open up to 6 parallel TCP connections per host.
  * **Asset Concatenation / Sprites:** Combining CSS/JS/images to minimize request counts.

---

### 2. HTTP/2 — Binary Multiplexing
* **Binary Framing Layer:** Replaces plaintext with discrete binary frames (`HEADERS`, `DATA`, `SETTINGS`).
* **Multiplexing:** Multiple requests and responses interleaved concurrently across a **single** TCP connection as independent streams. Solves HTTP application-level HoL blocking!
* **HPACK Header Compression:** Eliminates redundant header transfers by maintaining dynamic and static compression tables.
* **Server Push:** Server can push critical assets (e.g., `main.css`) to the client cache before the client explicitly asks.
* **The Catch — TCP-Level Head-of-Line Blocking:**  
  Because HTTP/2 runs over a single TCP stream, if one packet drops at the transport layer, TCP pauses **all** multiplexed streams until that missing packet is retransmitted.

---

### 3. HTTP/3 — QUIC over UDP
* **Transport via QUIC (UDP):** Replaces TCP with QUIC, a transport protocol built on top of UDP.
* **Independent Streams:** Streams in QUIC are truly isolated. If packet loss occurs on Stream 4, Streams 1, 2, and 3 continue unimpeded (eliminates TCP-level HoL blocking!).
* **Zero RTT Handshakes (0-RTT):** Integrates cryptographic and transport handshakes into one step. Re-connecting clients can send data immediately.
* **Connection Migration:** Connections are identified by a 64-bit **Connection ID**, not by IP:Port. Switching from Wi-Fi to cellular data does not drop active downloads or video streams!

---

### Deep Comparison Table

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
| :--- | :--- | :--- | :--- |
| **Underlying Transport** | TCP | TCP | QUIC (UDP) |
| **Data Format** | Text / ASCII | Binary Frames | Binary Frames |
| **Multiplexing** | ❌ No (Serial) | ✅ Yes (Streams over 1 TCP) | ✅ Yes (Native QUIC streams) |
| **Head-of-Line Blocking**| Application & Transport | Transport-level only (TCP) | ❌ None (Fully eliminated) |
| **Header Compression** | ❌ None | ✅ HPACK | ✅ QPACK (Out-of-order safe) |
| **TLS Integration** | Optional (TLS over TCP) | De-facto required (TLS 1.2+) | Mandatory (Built-in TLS 1.3) |
| **Handshake Latency** | 2–3 RTT (TCP + TLS) | 2–3 RTT (TCP + TLS) | 1 RTT (0-RTT on resumption) |
| **Connection Migration** | ❌ Fails on IP change | ❌ Fails on IP change | ✅ Seamless (Connection ID) |

---

## 🔹 9. HTTPS & TLS Architecture

HTTPS is HTTP transmitted over an encrypted **TLS (Transport Layer Security)** tunnel.

### Why HTTPS is Mandatory
1. **Confidentiality:** Data cannot be eavesdropped by intermediaries (ISPs, public Wi-Fi routers).
2. **Integrity:** Data cannot be modified or tampered with in transit.
3. **Authentication:** Proves the client is communicating with the legitimate domain owner via Digital Certificates signed by trusted **Certificate Authorities (CAs)**.

---

### TLS 1.3 Handshake Flow (1-RTT)

```text
Client                                                          Server
  │                                                               │
  │─── ClientHello ──────────────────────────────────────────────>│
  │    (Supported Ciphers + Key Share + SNI)                      │
  │                                                               │
  │<── ServerHello + Certificate + Server Key Share + Finished ───│
  │    (Server verifies identity, derives symmetric session key)  │
  │                                                               │
  │─── Finished + Encrypted Application Data (HTTP) ─────────────>│
  │<── Encrypted Application Data (HTTP) ─────────────────────────│
```

1. **ClientHello:** Client sends supported cipher suites, TLS version, Server Name Indication (SNI), and its Diffie-Hellman key share.
2. **ServerHello & Keys:** Server selects cipher suite, shares its public key share, and sends its signed X.509 Certificate.
3. **Session Key Derivation:** Both parties independently calculate the identical **symmetric session key** (AES-GCM or ChaCha20).
4. **Encrypted Communication:** All subsequent HTTP traffic is encrypted with the symmetric session key (fast and lightweight).

---

## 🔹 10. System Design Architectural Implications

```text
Client Requests
      │
      ▼
┌──────────────┐
│  Cloudflare  │  <- Edge CDN (SSL Termination, Static Caching, HTTP/3 to HTTP/2)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Reverse Proxy│  <- NGINX / ALB (L7 Routing, Rate Limiting, Header Validation)
│ (L7 Gateway) │
└──────┬───────┘
       │ Connection Pool (HTTP/1.1 or gRPC)
       ▼
┌──────────────┐
│ Microservice │  <- Backend Business Logic
└──────────────┘
```

### 1. SSL/TLS Termination at the Edge
* Terminate TLS at the CDN (Cloudflare) or Load Balancer (ALB).
* Offloads CPU-intensive cryptographic handshakes from application servers.
* Internal traffic behind the VPC can run over lightweight internal connections or mTLS.

### 2. Layer 4 vs Layer 7 Load Balancing
* **L4 (Transport Layer - TCP/UDP):** Routes raw packets based on IP:Port. Cannot inspect HTTP headers, paths, or cookies. High throughput, low latency.
* **L7 (Application Layer - HTTP):** Inspects HTTP path (`/api/v1/users` vs `/api/v1/video`), headers, and cookies. Enables intelligent routing, auth validation, and cookie-based sticky sessions.

### 3. Safe Retries with Idempotency Keys
* Distributed networks fail unpredictably.
* When retrying failed `POST` requests, client attaches an `Idempotency-Key: <UUID>`.
* Backend checks a fast distributed store (e.g., Redis):
  * If key exists and processed → return cached response without re-executing.
  * If key in flight → reject or wait.
  * If key new → process transaction and store result atomically.

---

## 🔥 System Design Interview Gotchas

* **301 Moved Permanently vs 302 Found:**  
  In a URL Shortener, returning **301** causes browsers to cache the destination URL locally. Subsequent clicks bypass your server entirely — great for latency, but **destroys link analytics**. Use **302** if you need to track every single click.
* **401 vs 403:**  
  `401` = Not logged in (Authentication missing). `403` = Logged in, but forbidden from accessing this resource (Authorization failure).
* **502 Bad Gateway vs 504 Gateway Timeout:**  
  `502` means the reverse proxy reached the upstream server, but it crashed or returned garbage. `504` means the upstream service or database was too slow and breached the gateway's timeout budget.
* **Why not always use HTTP/2 Server Push?**  
  Server push is difficult to coordinate with client caches. If the client already has the asset cached, pushed assets waste valuable bandwidth. Most modern architectures prefer `<link rel="preload">` tags or CDN edge caching instead.

---

## 🎯 One-Line Answer

> **HTTP** is the foundational stateless application protocol for client-server communication; modern systems leverage **persistent connections**, **HTTP/2 multiplexing**, and **HTTP/3 over QUIC** alongside proper caching headers, status code semantics, and idempotency guarantees to achieve low-latency, scalable, and resilient distributed architectures.
