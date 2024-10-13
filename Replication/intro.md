# Introduction
Replication means keeping the same data on multiple nodes which are connected over a network. A node which has the same copy of data is called a replica.

Reasons for Replication:
- To keep the data geographically close so as to serve incoming requests with lower latency.
- Fault Tolerance - To continue serving requests should some of the nodes go down.
- Increased throughput - To be able to handle a greater throughput that can be served from multiple replicas.

If the data does not change over time then replication is rather simple. 
However, this is not the case as multiple writes come into the system and we focus on creating replication strategies to be able to handle this without blocking the system (i.e violating availability criteria).

Based on the requirements of the application, there can be several replication strategies:
- [Single Leader Replication](./Single-Leader.md)
- [Multi-leader Replication](./Multi-leader.md)
- [Leaderless Replication](./Leaderless.md)

There are many tradeoffs that gets considered as to what would be the right replication strategy like if we have to go with Synchronous or Asynchronous replication and we will discuss that in greater detail
