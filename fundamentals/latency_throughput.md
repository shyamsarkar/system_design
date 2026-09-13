# ⏱️ Latency vs Throughput

## 1. Definitions

* **Latency:** The **time taken** to complete a single unit of work (from request to response).
  * **Focus:** Speed / Delay
  * **Units:** Milliseconds (ms), Seconds (s)
  * **Question:** *"How fast does a single request return?"*

* **Throughput:** The **number of units of work** processed within a given time period.
  * **Focus:** Capacity / Volume / Rate
  * **Units:** Requests Per Second (RPS), Queries Per Second (QPS), Megabits/sec (Mbps)
  * **Question:** *"How many requests can the system handle at once?"*

---

## 2. The Mental Model (Analogy)

```text
┌─────────────────────────────────────────────────────────────┐
│                     THE HIGHWAY ANALOGY                     │
├─────────────┬───────────────────────────────────────────────┤
│ Latency     │ Time taken for one car to travel from A to B  │
│ Throughput  │ Number of cars passing the toll gate per hour │
│ Bandwidth   │ Total number of lanes on the highway          │
└─────────────┴───────────────────────────────────────────────┘
```

> 💡 **Classic Example:** A truck full of hard drives driving across the country has **high latency** (days to arrive), but **huge throughput** (petabytes of data transferred).

---

## 3. Comparison

| Dimension | Latency | Throughput |
| :--- | :--- | :--- |
| **What it measures** | Delay / Duration of a single operation | Rate of completed operations over time |
| **Primary Metric** | Milliseconds (ms), Seconds (s) | RPS, QPS, TPS, Mbps/Gbps |
| **Target Goal** | As low as possible (Minimize) | As high as possible (Maximize) |
| **Key Bottlenecks** | Network distance (RTT), DB disk seeks, CPU processing | Network bandwidth, CPU cores, connection pool limits, concurrency |
| **Impact on User** | Perceived page load and API responsiveness | System availability under peak traffic load |
| **Optimization Techniques** | Caching, CDNs, connection reuse (HTTP/2, HTTP/3), indexing | Horizontal scaling, batching, load balancing, asynchronous queues |

---

## 4. Key Relationship

$$\text{Concurrency} = \text{Throughput} \times \text{Latency}$$

* Optimizing for throughput often increases latency (e.g., **batching** holds requests to process them in bulk, raising throughput but adding delay).
* Optimizing for lowest latency may reduce throughput (e.g., sending small individual requests immediately increases protocol overhead).
