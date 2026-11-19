# Consistency Models (Interview Ready)

## 1. Core Consistency Models

### 1.1 Strong Consistency (Linearizability)
Every read returns the value of the latest completed write, as if all operations happened atomically in a single global order.

- **Example:** After transferring money, reading the account balance must reflect the updated balance.
- **Use cases:** Banking transactions, inventory reservations, ownership records.
- **Trade-off:** May increase latency and reduce availability during network partitions, depending on the system's consistency guarantees.

### 1.2 Eventual Consistency
After a successful write, replicas may temporarily contain different values. However, if no new updates occur and replication continues successfully, all replicas will eventually converge to the same value.

- **Example:** You update your profile picture, but another user may temporarily see the old picture because their request reaches a replica that has not received the update.
- **Use cases:** Social media feeds, product reviews, analytics, search indexes, distributed counters.
- **Trade-off:** Reads may return stale (outdated) data temporarily.

### 1.3 Read-After-Write Consistency
Once a client successfully writes data, subsequent reads by that client are guaranteed to reflect that write or a newer write.

- **Example:** After updating your profile name, you should immediately see the updated name when you refresh your profile.
- **Use cases:** User profiles, settings, user-generated content.
- **Trade-off:** The system may need session tracking, sticky reads, or routing reads to an up-to-date replica.

### 1.4 Monotonic Read Consistency
Once a client has observed a particular version of data, subsequent reads by that client will never return an older version.

- **Example:** You see your account balance as ₹5,000. Your next read should not show an older balance of ₹4,000.
- **Use cases:** User dashboards, distributed applications, replicated databases.
- **Trade-off:** Requires maintaining read-version information or routing requests appropriately.

### 1.5 Monotonic Write Consistency
Writes from the same client are applied in the order in which that client issued them.

- **Example:** A user changes their username and then updates their profile description. The system must preserve the order of these operations.
- **Use cases:** User sessions, distributed applications, collaborative systems.
- **Trade-off:** May require request sequencing or session affinity.

---

## 2. Choosing the Right Consistency Model

| Use Case | Preferred Consistency |
|---|---|
| Bank account balances | Strong |
| Payment processing | Strong for financial invariants |
| Inventory reservation | Strong |
| User profile updates | Read-after-write |
| Social media feeds | Eventual |
| Search indexes | Eventual |
| Analytics dashboards | Eventual |
| Product recommendations | Eventual |
| User session data | Read-after-write / Monotonic reads |

**Important:** Choose consistency based on business requirements, not simply because one model is faster.

For example, a social media feed can tolerate a slightly outdated post, but a banking system cannot safely allow two concurrent withdrawals to violate the account balance invariant.

---

## 3. Important Distinctions

### Consistency vs. Correctness
- **Consistency:** Defines what values a read is allowed to return relative to writes.
- **Correctness:** Means the application follows its business rules and produces valid results.

### Consistency vs. Durability
- **Consistency:** Determines what state readers can observe.
- **Durability:** Ensures committed data survives failures.

### Consistency vs. Availability
- **Consistency:** Determines whether reads return values satisfying the consistency guarantee.
- **Availability:** Determines whether the system continues responding to requests.

### Consistency vs. Replication
- **Replication:** Maintains copies of data across multiple nodes.
- **Consistency:** Defines how those copies behave when reads and writes occur.

Replication does not automatically guarantee strong consistency.

---

## 4. Consistency and Network Partitions (CAP Theorem)

During a network partition, distributed systems face a trade-off between:

- **Consistency (C):** Every read receives the latest successful write, according to the system's consistency guarantee.
- **Availability (A):** Every request to a non-failed node receives a response, even if that response cannot contain the latest data.
- **Partition Tolerance (P):** The system continues operating despite communication failures between nodes.

**Interview point:** When a network partition occurs, a distributed system must choose whether to reject or delay some requests to preserve consistency, or continue responding with potentially stale data to preserve availability.

---

## 5. Interview Checklist

When designing a distributed system, ask:

1. **Business invariant:** What must never happen? (e.g., no double spending).
2. **Consistency requirement:** Do we need strong, eventual, or session-level consistency?
3. **Read/write path:** Does the request go to the primary database or a replica?
4. **Replication strategy:** Is replication synchronous or asynchronous?
5. **Stale reads:** Can users temporarily see outdated data?
6. **Conflict resolution:** What happens if two nodes update the same record?
7. **Failure handling:** What happens during network partitions?
8. **User experience:** Should we show stale data, retry, or return an error?

---

## 6. Interview Summary

- **Strong consistency:** Reads always reflect the latest completed write according to the global ordering guarantee.
- **Eventual consistency:** Replicas converge when updates stop and replication succeeds.
- **Read-after-write:** A client sees its own successful writes.
- **Monotonic reads:** A client never moves backward to an older version of data.
- **Monotonic writes:** A client's writes are applied in their original order.

**Golden rule:** Use the strongest consistency guarantee required by the business invariant, not necessarily the strongest guarantee everywhere.