# System Design Roadmap

# High-Level Design (HLD)

## Step 1: Fundamentals

- **How does the Internet work?**
  - DNS
  - IP
  - TCP/IP basics
  - Routing basics

- **HTTP & HTTPS**
  - HTTP request/response
  - HTTP Methods (HTTP Verbs)
  - Status codes
  - Cookies
  - Headers
  - Keep-alive
  - HTTP/1.1 vs HTTP/2 vs HTTP/3

- **Processes vs Threads**
- **Concurrency vs Parallelism**
- **Horizontal vs Vertical Scaling**
- **Stateless vs Stateful Applications**
- **Latency vs Throughput**
- **Availability, Reliability & Scalability**
- **Serverful vs Serverless** *(Optional — Later)*


---

## Step 2: Load Balancing & Traffic Management

- **Reverse Proxy**
- **Forward Proxy** *(Optional — Later)*
- **Load Balancer**
- **Layer 4 vs Layer 7 Load Balancing**
- **Load Balancing Algorithms**
  - Round Robin
  - Least Connections
  - IP Hash

- **Health Checks**
- **Sticky Sessions**
- **Rate Limiting**
  - Fixed Window
  - Sliding Window
  - Token Bucket
  - Leaky Bucket

- **Consistent Hashing**


---

## Step 3: Databases & Storage

- **SQL vs NoSQL**
- **Relational Database Fundamentals**
- **Schema Design**
- **Normalization vs Denormalization**
- **Indexes**
  - B-Tree
  - Hash
  - Composite Index
  - Covering Index *(Optional — Later)*

- **Query Optimization**
- **Transactions & ACID**
- **Isolation Levels**
- **MVCC**
- **Locks & Deadlocks**
- **Optimistic vs Pessimistic Locking**
- **Hot Rows / Contention**
- **Database Replication**
  - Primary/Replica
  - Read Replicas
  - Replication Lag
  - Read/Write Splitting

- **Database Partitioning**
- **Sharding**
- **Hot Partitions / Hotspots**
- **Data Locality**
- **Data Migration**
- **Online Schema Changes**
- **Connection Pooling**
- **Write-Ahead Logging (WAL)** *(Conceptual)*


---

## Step 4: Caching & CDN

- **Why Caching?**
- **Cache-aside**
- **Write-through**
- **Write-back**
- **Write-around**
- **Cache Invalidation**
- **TTL**
- **Cache Eviction**
  - LRU
  - LFU

- **Cache Stampede / Thundering Herd**
- **Hot Keys**
- **Distributed Caching**
- **Redis vs Memcached**
- **CDN**
- **Edge Caching**


---

## Step 5: Object & File Storage

- **Block vs File vs Object Storage**
- **Object Storage**
  - S3
  - GCS

- **Large File Uploads**
- **Multipart Upload**
- **Pre-signed URLs**
- **File Metadata**
- **Hot vs Cold Storage**
- **Storage Lifecycle Policies**

### Optional — Later

- **Distributed File Systems**
  - HDFS
  - Ceph
  - GlusterFS
- **Advanced Storage Architecture**


---

## Step 6: Architecture Patterns

- **Client-Server Architecture**
- **Layered Architecture**
- **N-Tier Architecture**
- **Monolith**
- **Modular Monolith**
- **Microservices**
- **Service Boundaries**
- **Database-per-Service**
- **Shared Database**
- **API Gateway**
- **Backend for Frontend (BFF)**
- **Event-Driven Architecture**
- **Pub/Sub**
- **Pipe & Filter**
- **Strangler Fig Pattern**
- **Sidecar Pattern**

### Optional — Later

- **Advanced Microservice Deployment Patterns**
- **Service Mesh Architecture**


---

## Step 7: Distributed Systems Fundamentals

- **CAP Theorem**
- **Consistency**
  - Strong Consistency
  - Eventual Consistency
  - Read-after-write Consistency

- **Replication**
- **Leader / Follower**
- **Quorum**
- **Leader Election**
- **Consensus**
  - Raft — high level
  - Paxos — high level

- **Distributed Locks**
- **Distributed Transactions**
  - Two-Phase Commit — high level
  - Saga Pattern

- **Network Partitions**
- **Split Brain**
- **Failure Detection**
- **Clock & Ordering Problems**
  - Logical Clocks — basic understanding

### Optional — Later

- **Vector Clocks**
- **Lamport Clocks — deeper study**
- **Byzantine Fault Tolerance**
- **Advanced Consensus Algorithms**


---

## Step 8: API & Service Communication

- **REST**
- **gRPC**
- **API Gateway**
- **Service Discovery**
- **Synchronous vs Asynchronous Communication**
- **Request Timeouts**
- **Connection Pooling**
- **API Versioning**
- **Pagination**
- **Idempotency**
- **Idempotency Keys**
- **API Rate Limiting**
- **Request Deduplication**
- **Retry Semantics**

### Optional — Later

- **GraphQL**
- **GraphQL Federation**
- **Advanced Service Mesh Concepts**


---

## Step 9: Message Queues & Event-Driven Systems

- **Why Message Queues?**
- **Producer / Consumer**
- **Queue vs Pub/Sub**
- **Kafka**
- **RabbitMQ**
- **SQS — conceptual**
- **Message Ordering**
- **Message Delivery Guarantees**
  - At-most-once
  - At-least-once
  - Exactly-once — practical limitations

- **Retries**
- **Dead Letter Queues**
- **Consumer Groups**
- **Backpressure**
- **Batch Processing**
- **Chunking**
- **Fan-out**
- **Fan-in**
- **Scatter-Gather**
- **Idempotent Consumers**
- **Event-Driven Architecture**
- **Outbox Pattern**
- **Inbox Pattern**

### Optional — Later

- **Event Sourcing**
- **CQRS**
- **Kafka Streams**
- **Apache Flink**
- **Advanced Stream Processing**
- **Advanced Event-Driven Architecture**


---

## Step 10: Reliability & Fault Tolerance

- **Timeouts**
- **Retries**
- **Exponential Backoff**
- **Jitter**
- **Circuit Breaker**
- **Bulkheads**
- **Health Checks**
- **Graceful Degradation**
- **Failover**
- **Cascading Failures**
- **Load Shedding**
- **Backpressure**
- **Idempotency**
- **Duplicate Request Handling**
- **Disaster Recovery**
- **Backup & Restore**
- **RPO**
- **RTO**


---

## Step 11: Scalability & Architecture

- **Monolith**
- **Modular Monolith**
- **Microservices**
- **Service Boundaries**
- **Database-per-Service**
- **Shared Database**
- **Microservice Communication**
- **Distributed Transactions**
- **Event-Driven Architecture**
- **Strangler Fig Pattern**
- **Containerization**
  - Docker

### Optional — Later

- **Kubernetes**
- **Container Orchestration**
- **Service Mesh**
- **Sidecar Pattern**
- **Advanced Microservice Deployment Strategies**


---

## Step 12: Cloud Architecture Fundamentals

- **Regions**
- **Availability Zones**
- **Multi-AZ Architecture**
- **Multi-Region Architecture**
- **Autoscaling**
- **Object Storage**
- **Managed Databases**
- **Managed Queues**
- **CDN**
- **DNS-based Traffic Routing**
- **Disaster Recovery Across Regions**

### Optional — Later

- **Advanced Cloud Networking**
- **Advanced Kubernetes**
- **Infrastructure as Code**
- **Multi-Cloud Architecture**


---

## Step 13: Search & Specialized Infrastructure

- **Search Engines**
  - Elasticsearch / OpenSearch

- **Inverted Index**
- **Full-text Search**
- **Autocomplete**
- **Geospatial Search**
- **Distributed Counters**
- **Unique ID Generation**
  - UUID
  - Snowflake-style IDs

### Optional — Later

- **Recommendation Systems**
- **Search Ranking**
- **Advanced Geospatial Systems**
- **Distributed Search Internals**


---

## Step 14: Networking for Distributed Applications

- **TCP vs UDP**
- **HTTP/1.1 vs HTTP/2 vs HTTP/3**
- **WebSockets**
- **Long Polling**
- **Server-Sent Events**

### Optional — Later

- **WebRTC**
- **Video Streaming Architecture**
- **Video Encoding / Transcoding**
- **Adaptive Bitrate Streaming**
- **QUIC Internals**
- **Advanced Network Protocols**


---

## Step 15: Observability

- **Logging**
- **Metrics**
- **Tracing**
- **Distributed Tracing**
- **Correlation IDs**
- **Health Checks**
- **Monitoring**
- **Alerting**
- **SLI**
- **SLO**
- **SLA**
- **P50 / P95 / P99**
- **Error Rate**
- **Throughput**
- **Latency**

### Optional — Later

- **Anomaly Detection**
- **Advanced Distributed Tracing**
- **Log Aggregation Internals**
- **Advanced Observability Platforms**


---

## Step 16: Security

- **Authentication vs Authorization**
- **Session-based Authentication**
- **Token-based Authentication**
- **JWT**
- **OAuth 2.0**
- **OpenID Connect**
- **SSO**
- **RBAC**
- **ACL**
- **Encryption**
  - At Rest
  - In Transit

- **TLS**
- **Secrets Management**
- **API Security**
- **Rate Limiting**
- **OWASP Basics**
  - XSS
  - CSRF
  - SQL Injection
  - SSRF

- **Password Hashing**

### Optional — Later

- **Advanced OAuth/OIDC**
- **mTLS**
- **Zero Trust Architecture**
- **Advanced Secrets Management**
- **Advanced Cryptography**
- **Web Application Firewall (WAF)**
- **Advanced Security Architecture**


---

## Step 17: Distributed Data & Processing Patterns

- **Idempotency Keys**
- **Deduplication**
- **Distributed Counters**
- **Distributed Sequences**
- **Batch Processing**
- **Chunking**
- **Fan-out**
- **Fan-in**
- **Scatter-Gather**
- **Work Queues**
- **Backpressure**
- **Hot Keys**
- **Hot Rows**
- **Data Locality**
- **Request Coalescing**


---

## Step 18: System Design Trade-offs

- **Consistency vs Availability**
- **Latency vs Throughput**
- **Read vs Write Optimization**
- **SQL vs NoSQL**
- **Synchronous vs Asynchronous**
- **Push vs Pull**
- **Polling vs WebSockets**
- **Strong vs Eventual Consistency**
- **Vertical vs Horizontal Scaling**
- **Caching vs Database Load**
- **Accuracy vs Latency**
- **Storage Cost vs Performance**
- **Availability vs Cost**
- **Complexity vs Scalability**
- **Build vs Buy**
- **Consistency vs Latency**
- **Performance vs Simplicity**


---

## Step 19: Back-of-the-Envelope Calculations

- **QPS Estimation**
- **Peak QPS**
- **Requests/day → QPS**
- **Storage Estimation**
- **Storage Growth**
- **Bandwidth Estimation**
- **Memory Estimation**
- **Replication Storage**
- **Cache Size Estimation**
- **Capacity Planning**
- **Latency Budget**
- **Server Capacity Estimation**
- **Availability Calculations**
- **Replication Factor**
- **Cost Estimation**

### Basic Units

- KB
- MB
- GB
- TB
- Requests/second
- Bytes/second
- GB/day
- TB/year


---

## Step 20: Real System Design Practice

### Beginner

- **✅ URL Shortener**
- **Pastebin**
- **✅ Rate Limiter**
- **✅ Notification Service**
- **File Upload Service**
- **Distributed ID Generator**

### Intermediate

- **Chat Application**
- **Food Delivery System**
- **Movie Ticket Booking**
- **E-commerce System**
- **✅ Payment System**
- **Ride Booking System**
- **Google Drive / Dropbox**
- **Social Media Feed**
- **Job Scheduling System**
- **Restaurant Ordering System**
- **Multi-Tenant SaaS System**

### Advanced

- **YouTube**
- **Netflix**
- **Instagram**
- **WhatsApp**
- **Uber**
- **Twitter/X**
- **Zoom**
- **Airbnb / Booking.com**
- **Distributed Logging System**
- **Large-scale Notification System**
- **Distributed Job Queue**
- **Large-scale Analytics System**
- **Multi-Region SaaS System**

### System Design Process

For every problem:

1. **Requirements**
2. **Functional Requirements**
3. **Non-functional Requirements**
4. **Scale Estimation**
5. **API Design**
6. **Data Model**
7. **High-level Architecture**
8. **Database Choice**
9. **Caching**
10. **Asynchronous Processing**
11. **Scaling**
12. **Bottleneck Identification**
13. **Failure Scenarios**
14. **Consistency Requirements**
15. **Security**
16. **Observability**
17. **Trade-offs**
18. **Cost Considerations**

### Failure Analysis

For every important component ask:

- What happens if it goes down?
- What happens if the network fails?
- What happens if the request is duplicated?
- What happens if a dependency becomes slow?
- What happens if traffic suddenly increases?
- What happens if data becomes inconsistent?
- How do we recover?


---

## Step 21: System Design Interview Skills

- **Requirement Clarification**
- **Identifying Constraints**
- **Making Explicit Assumptions**
- **Estimating Scale Quickly**
- **Choosing Reasonable Defaults**
- **Designing the Simplest Working Architecture First**
- **Identifying Bottlenecks**
- **Explaining Trade-offs**
- **Failure Analysis**
- **Capacity Planning**
- **Drawing Architecture Clearly**
- **Explaining Architecture Verbally**
- **Handling Follow-up Questions**
- **Defending Design Decisions**
- **Knowing When NOT to Use a Technology**
- **Prioritizing Requirements**
- **Iteratively Scaling the Design**


---

# Low-Level Design (LLD)

## Step 1: OOP & Object Design

- **Encapsulation**
- **Abstraction**
- **Composition vs Inheritance**
- **Polymorphism**
- **Interfaces**
- **SOLID Principles**
- **Dependency Injection**
- **Cohesion & Coupling**


---

## Step 2: Design Patterns

### Creational

- **Factory**
- **Abstract Factory** *(Optional — Later)*
- **Builder**
- **Singleton**
  - Understand why it can be problematic

### Structural

- **Adapter**
- **Decorator**
- **Facade**
- **Proxy**
- **Composite** *(Optional — Later)*

### Behavioral

- **Strategy**
- **Observer**
- **Command**
- **State**
- **Template Method** *(Optional — Later)*
- **Chain of Responsibility** *(Optional — Later)*

### Pattern Learning Strategy

For every pattern, understand:

- **Problem**
- **Pattern**
- **Why use it?**
- **When not to use it?**
- **Alternative approaches**


---

## Step 3: Concurrency & Thread Safety

- **Processes vs Threads**
- **Concurrency vs Parallelism**
- **Race Conditions**
- **Critical Sections**
- **Mutex / Locks**
- **Read/Write Locks**
- **Semaphores**
- **Deadlocks**
- **Thread Safety**
- **Atomic Operations**
- **Producer–Consumer**
- **Thread Pools**
- **Async Programming**
- **Synchronization**


---

## Step 4: API & Code Design

- **API Design**
- **Request/Response Modeling**
- **Validation**
- **Error Handling**
- **API Versioning**
- **Pagination**
- **Idempotency**
- **Extensibility**
- **Dependency Injection**
- **Configuration Management**
- **Clean Code**
- **DRY**
- **SRP**
- **Avoiding God Classes**
- **Separation of Concerns**


---

## Step 5: Database Design

- **Schema Design**
- **Normalization**
- **Denormalization**
- **Relationships**
- **Constraints**
- **Indexes**
- **Transactions**
- **Locking**
- **Database Migrations**
- **Optimistic vs Pessimistic Locking**


---

## Step 6: UML & Design Modeling

- **Class Diagrams**
- **Sequence Diagrams**
- **State Diagrams**
- **Component Diagrams**
- **Mapping Design → Code**

### Optional — Later

- **Advanced UML**
- **Deployment Diagrams**
- **Activity Diagrams**

> Focus on communicating design clearly rather than becoming a UML specialist.


---

## Step 7: Testing

- **Unit Testing**
- **Integration Testing**
- **Mocking**
- **Test Doubles**
- **Testability in Design**

### Optional — Later

- **Contract Testing**
- **Property-based Testing**
- **Mutation Testing**
- **Advanced Test Architecture**


---

## Step 8: LLD Practice Problems

### Beginner

- **Tic-Tac-Toe**
- **Parking Lot**
- **Elevator**
- **Library Management**

### Intermediate

- **Splitwise**
- **Movie Ticket Booking**
- **Food Delivery**
- **Notification System**
- **ATM**
- **Vending Machine**

### Advanced

- **Rate Limiter**
- **Logging Framework**
- **Task Scheduler**
- **Message Queue**
- **Payment System**
- **File Storage System**


---

# Execution Strategy

## Core Learning Loop

**Learn → Understand Why → Implement a Small Version → Design a Large-Scale Version → Find Bottlenecks → Redesign → Explain Aloud**

### Rules

- **Learn → Implement a small version → Design a large-scale version**
- **Understand "Why" before "What"**
- **For every technology, understand the problem it solves**
- **For every architecture, understand its trade-offs**
- **Use real-world systems as examples**
- **Practice designing systems from requirements**
- **Use your own projects to practice scaling**
- **Design the simplest architecture first**
- **Only add complexity when a requirement justifies it**
- **Practice failure scenarios, not just the happy path**
- **Practice explaining your design aloud**

## Recommended Learning Ratio

- **30% → Learn concepts**
- **20% → Implement small examples**
- **50% → Design systems and solve case studies**


## Recommended Mindset

Don't learn system design as:

> "I need to know Redis, Kafka, Kubernetes, Cassandra, etc."

Learn it as:

> **"I have a problem. What constraints does it have? What architecture solves it? What are the trade-offs? What can fail?"**


## Before Designing Any System

Ask:

- Who are the users?
- What operations are required?
- How much traffic?
- How much data?
- What latency is expected?
- What availability is required?
- What consistency is required?
- What are the failure scenarios?
- What are the security requirements?
- What are the cost constraints?
- What are the scaling requirements?

Then define:

- **Functional Requirements**
- **Non-functional Requirements**
- **Scale**
- **Constraints**

Only then design the architecture.


## For Every Technology

Understand:

1. **What problem does it solve?**
2. **Why is the problem difficult?**
3. **How does the technology solve it?**
4. **What are its advantages?**
5. **What are its disadvantages?**
6. **When should I use it?**
7. **When should I NOT use it?**
8. **What happens when it fails?**
9. **What are the alternatives?**
10. **What trade-offs does it introduce?**


## For Every System Design Problem

Start simple:

```text
Requirements
    ↓
Scale Estimation
    ↓
API Design
    ↓
Data Model
    ↓
Basic Architecture
    ↓
Identify Bottlenecks
    ↓
Scale the Bottleneck
    ↓
Handle Failures
    ↓
Add Reliability
    ↓
Add Observability
    ↓
Security
    ↓
Trade-offs
    ↓
Cost
