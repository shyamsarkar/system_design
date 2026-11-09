# Redis Performance Interview Question

## Why is Redis so fast even though it is single-threaded?

Redis keeps data in memory, uses efficient data structures, and uses an event loop with non-blocking I/O to serve many client connections without a thread per connection. Sequential command execution also avoids much of the locking and synchronization overhead that concurrent command execution would require.

The trade-off is that a slow or CPU-intensive command can delay other clients because commands are processed sequentially. Redis can also be limited by CPU, memory, network bandwidth, and serialization or large-value operations. Modern Redis versions can use I/O threads for some network work, while command execution remains primarily single-threaded.
