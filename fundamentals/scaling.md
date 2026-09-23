# Horizontal vs Vertical Scaling (Interview Ready)

## 1. Definitions

- **Vertical scaling:** give one machine more CPU, memory, storage, or network capacity.
- **Horizontal scaling:** add more machines or service instances and distribute work among them.

## 2. Comparison

| Dimension | Vertical | Horizontal |
| :--- | :--- | :--- |
| Simplicity | Simple at first | Requires coordination and routing |
| Upper bound | Hardware limit | Usually much higher |
| Failure domain | Large single-instance impact | Smaller per-instance impact |
| State | Easy to keep local | Must externalize or partition state |
| Cost | Larger machines can be expensive | More commodity capacity |

## 3. Interview Decision

Start vertically when the workload is small or a stateful dependency is difficult to distribute. Move horizontally when availability, traffic growth, or a single-machine limit requires it.

Horizontal scaling usually requires stateless application instances, load balancing, shared storage, partitioning, replication, and idempotent operations.

## 4. Common Bottlenecks

- Database write capacity
- Hot keys or hot partitions
- Connection limits
- Coordination and leader contention
- Uneven traffic distribution

## 5. Interview Checklist

State the bottleneck, estimate capacity per instance, define the scaling trigger, explain how state is handled, and describe rebalancing and failure behavior.
