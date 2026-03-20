# System Design Roadmap

## High-Level Design (HLD)

### Step 1: Fundamentals
- **Serverless vs Serverful**
- **Horizontal vs Vertical Scaling**
- **Threads & Processes**
- **Memory Management (Paging, Caching basics)**
- **How does the Internet work?**

---

### Step 2: Databases & Storage
- **SQL vs NoSQL Databases**
- **In-memory Databases**
- **Data Replication & Migration**
- **Data Partitioning & Sharding**
- **Indexing (B-Tree, Hash Index)**
- **Query Optimization**
- **Blob/Object Storage (S3, GCS)**
- **File Systems (HDFS, Distributed FS)**
- **Hot vs Cold Storage**

---

### Step 3: Distributed Systems Basics
- **CAP Theorem**
- **Consistency & Its Levels**
- **Isolation Levels**
- **Leader Election**
- **Consensus Algorithms (Raft, Paxos - high level)**
- **Distributed Locks**

---

### Step 4: Caching
- **What is Cache? (Redis, Memcached)**
- **Write Policies:**
  - Write-back  
  - Write-through  
  - Write-around  
- **Replacement Policies:**
  - LFU  
  - LRU  
  - Segmented LRU  
- **Cache Invalidation**
- **Content Delivery Networks (CDNs)**

---

### Step 5: Networking
- **TCP vs UDP**
- **HTTP (1/2/3) & HTTPS**
- **WebSockets**
- **WebRTC & Video Streaming**

---

### Step 6: API & Service Communication
- **API Gateway**
- **REST vs gRPC**
- **Service Discovery**

---

### Step 7: Load Balancers
- **Load Balancing Algorithms (Stateless & Stateful)**
- **Consistent Hashing**
- **Proxy & Reverse Proxy**
- **Rate Limiting**

---

### Step 8: Message Queues & Data Processing
- **Asynchronous Processing (Kafka, RabbitMQ)**
- **Publisher–Subscriber Model**
- **Stream Processing (Kafka Streams, Flink basics)**
- **Batch vs Stream Processing**

---

### Step 9: Fault Tolerance & Reliability
- **Retries & Exponential Backoff**
- **Circuit Breaker**
- **Idempotency**
- **Graceful Degradation**
- **Avoiding Cascading Failures**

---

### Step 10: Monoliths vs Microservices
- **Why Microservices?**
- **Migration to Microservices**
- **Containerization (Docker)**

---

### Step 11: Monitoring & Logging
- **Logging Events**
- **Monitoring Metrics**
- **Alerting Systems**
- **Anomaly Detection**

---

### Step 12: Security
- **Authentication vs Authorization**
- **JWT & Token-based Authentication**
- **SSO & OAuth**
- **Access Control Lists (ACL)**
- **Encryption (At Rest & In Transit)**
- **Basic OWASP (CSRF, XSS)**

---

### Step 13: System Design Trade-offs
- **Consistency vs Availability**
- **SQL vs NoSQL**
- **Push vs Pull Architecture**
- **Memory vs Latency**
- **Throughput vs Latency**
- **Accuracy vs Latency**

---

### Step 14: Back-of-the-envelope Calculations
- **QPS Estimation**
- **Storage Estimation**
- **Bandwidth Calculation**

---

### Step 15: Practice Problems
- **YouTube**
- **Twitter**
- **WhatsApp**
- **Uber**
- **Amazon**
- **Dropbox / Google Drive**
- **Netflix**
- **Instagram**
- **Zoom**
- **Booking.com / Airbnb**

---

## Low-Level Design (LLD)

### Step 1: Object-Oriented Programming (OOP)
- **Encapsulation**
- **Abstraction**
- **Inheritance**
- **Polymorphism**
- **SOLID Principles**

---

### Step 2: Design Patterns
- **Creational: Singleton, Factory, etc.**
- **Structural: Proxy, Bridge, etc.**
- **Behavioral: Strategy, Command, Observer, etc.**

---

### Step 3: Concurrency & Thread Safety
- **Thread-safe Design**
- **Locking Mechanisms**
- **Producer–Consumer Problem**
- **Race Conditions & Synchronization**

---

### Step 4: UML & Design Modeling
- **Class Diagrams**
- **Sequence Diagrams**
- **Mapping Design to Code**

---

### Step 5: APIs & Code Design
- **API Design**
- **Request/Response Modeling**
- **Versioning & Extensibility**
- **Clean Code Principles (DRY, SRP)**
- **Avoiding God Classes**
- **Error Handling Strategy**
- **Configuration Management**

---

### Step 6: Database Design (LLD)
- **Schema Design**
- **Normalization vs Denormalization**
- **Indexing in Practice**

---

### Step 7: Testing
- **Unit Testing**
- **Mocking**
- **Integration Testing Basics**

---

### Step 8: Common LLD Problems
- **Design Tic-Tac-Toe or Chess**
- **Design Splitwise**
- **Design a Parking Lot**
- **Design an Elevator System (Multiple Lifts)**
- **Design a Notification System**
- **Design a Food Delivery App**
- **Design a Movie Ticket Booking System**
- **Design a URL Shortener**
- **Design a Logging Framework**
- **Design a Rate Limiter**

---

## Execution Strategy
- Learn → Implement small version → Design large-scale system
- Revise each topic with 1 real-world example
- Focus on "Why" over "What"
- Practice consistently with real systems