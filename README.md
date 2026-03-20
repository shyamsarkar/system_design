# System Design Roadmap

## High-Level Design (HLD)

### Step 1: Fundamentals
- **Serverless vs Serverful**
- **Horizontal vs Vertical Scaling**
- **What are Threads?**
- **What are Pages?**
- **How does the Internet work?**

### Step 2: Databases
- **SQL vs NoSQL Databases**
- **In-memory Databases**
- **Data Replication & Migration**
- **Data Partitioning**
- **Sharding**

### Step 3: Consistency vs Availability
- **Data Consistency & Its Levels**
- **Isolation & Its Levels**
- **CAP Theorem**

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
- **Content Delivery Networks (CDNs)**

### Step 5: Networking
- **TCP vs UDP**
- **HTTP (1/2/3) & HTTPS**
- **WebSockets**
- **WebRTC & Video Streaming**

### Step 6: Load Balancers
- **Load Balancing Algorithms (Stateless & Stateful)**
- **Consistent Hashing**
- **Proxy & Reverse Proxy**
- **Rate Limiting**

### Step 7: Message Queues
- **Asynchronous Processing (Kafka, RabbitMQ)**
- **Publisher–Subscriber Model**

### Step 8: Monoliths vs Microservices
- **Why Microservices?**
- **Single Point of Failure**
- **Avoiding Cascading Failures**
- **Containerization (Docker)**
- **Migration to Microservices**

### Step 9: Monitoring & Logging
- **Logging Events**
- **Monitoring Metrics**
- **Anomaly Detection**

### Step 10: Security
- **Token-based Authentication**
- **SSO & OAuth**
- **Access Control Lists & Rule Engines**
- **Encryption**

### Step 11: System Design Trade-offs
- **Push vs Pull Architecture**
- **Consistency vs Availability**
- **SQL vs NoSQL**
- **Memory vs Latency**
- **Throughput vs Latency**
- **Accuracy vs Latency**

### Step 12: Practice Problems
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

### Step 2: Design Patterns
- **Creational: Singleton, Factory, etc.**
- **Structural: Proxy, Bridge, etc.**
- **Behavioral: Strategy, Command, Observer, etc.**

### Step 3: Concurrency & Thread Safety
- **Thread-safe Injection**
- **Locking Mechanisms**
- **Producer–Consumer Problem**
- **Race Conditions & Synchronization**

### Step 4: UML Diagrams

### Step 5: APIs
- **API Design**
- **Request/Response Modeling**
- **Versioning & Extensibility**
- **Clean Code Principle**
  - DRY  
  - SRP  
- **Avoiding God Classes**

### Step 6: Common LLD Problems
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

## Notes
- **Follow HLD → then LLD for better understanding**
- **Focus on trade-offs and real-world constraints**
- **Practice consistently with real systems**