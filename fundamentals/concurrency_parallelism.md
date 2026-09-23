# Concurrency vs Parallelism (Interview Ready)

## 1. Core Difference

- **Concurrency:** multiple tasks make progress during the same period; execution may be interleaved.
- **Parallelism:** multiple tasks execute at the same instant on multiple cores or workers.

Concurrency is about structure and coordination. Parallelism is about simultaneous execution.

## 2. Common Models

| Model | Useful for | Main risk |
| :--- | :--- | :--- |
| Threads | CPU or mixed workloads | Shared-state races |
| Async event loop | Many I/O-bound requests | Blocking the event loop |
| Worker pool | Bounded background work | Queue growth and saturation |
| Processes | Isolation and CPU parallelism | IPC and memory overhead |

## 3. Interview Trade-offs

- I/O-bound work benefits from asynchronous I/O or a sufficiently sized worker pool.
- CPU-bound work needs parallel workers, native execution, or separate processes.
- Unbounded concurrency usually moves the bottleneck to memory, connections, or a downstream dependency.
- Bound concurrency with queues, semaphores, or worker pools and expose saturation metrics.

## 4. Failure Modes

- Race conditions and lost updates
- Deadlocks and lock contention
- Starvation and unfair scheduling
- Queue buildup and backpressure failure
- Oversubscription and context-switch overhead

## 5. Interview Checklist

Explain whether the workload is CPU- or I/O-bound, identify the execution model, define the concurrency limit, and describe cancellation, timeouts, ordering, and shutdown behavior.
