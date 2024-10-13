# Leader - Follower / Active - Passive / Master - Slave Replication

## Working Principle
1. One of the Replicas is designated as the leader. This happens via a leader election algorithm (which will be discussed when we come to consistency).
   When the client wants to perform a write request, their requests will be sent to the leader which will then apply the changes first in its local storage.
2. The other replicas are called as followers / slaves / secondaries. Whenever the leader writes a new data to its local storage, it also sends a change log
   to all the other replicas (In MySQL, it is called Binlog and in PostgreSQL, it is called Log sequence number) called the replication log or change stream.
   Each follower takes the log from the leader and applies in the order in which they were processed on the leader.
3. When a client wants to query from the Database, it can either query from the Leader or Followers. However, writes are only accepted on the leader.

![Leader Based Replication](https://notes.shichao.io/dda/figure_5-1.png)

## Synchronous vs Asynchronous vs Semi-Synchronous Replication
An important aspect is to understand if the replication is happening Synchronously or Asynchronously. 

- **Synchronous Replication** - Unless all the configured followers acknowledge the change from the leader, the write will not be committed.
  - *Advantages*
    - It is guaranteed that the replica has the upto-date copy and is consistent with the leader replica.
    - If the leader fails, we can guarantee that the updates are not lost.
  - *Disadvantages*
    - If the synchronous replica / follower is not able to respond for some reason (system crash or network failure), then writes cannot be processed.
      This means that the leader is blocked until the followers respond with an ack.
      
  A purely synchronous replication strategy is disastrous because of the reason that if any one node fails, the entire distributed system will come to a halt.
  
- **Asynchronous Replication** - The leader can continue processing the writes from the client without explicitly receiving the Ack from the followers.
  The change is sent asynchronously and eventually changes will be written.
    - *Disadvantages*
      - If the leader fails and is in a non recoverable state, all writes not processed until that time will be lost. Hence there is no guarantee of durability.
        
While the data durability looks to be an issue, this asynchronous system of replication is widely used. If the application reads from an asynchronous follower, it
may see outdated information if the follower is lagging behind. This leads to inconsistencies between the data observed. Not all writes would have been reflected
at the same time yielding different results. If the writes at some point stop coming to the disk, the followers will *eventually* catch up and become consistent.
This aspect of consistency is also called **eventual consistency**. Some applications are just fine with this level of consistency. 
Here are some of the illustration mechanism:

Consider a situation where there are two comments made concurrently to a post on instagram: 
- How does it matter if the comment 1 came first and comment 2 came second or vice versa. At the end both the comments arrived.
- It is not necessary that all users in instagram needs to get the comment at the same time. It is still perfectly fine even if there is a lag of several minutes.

While purely synchronous systems are not desirable, due to data durability issues, a purely asynchronous system may also pose significant problems.
Hence, the hybrid approach is what is used today and is called **semisynchronous replication**.

Here the idea is that a single follower will be made a synchronous follower and rest of the followers will be made asynchronous.
Should a synchronous follower fail, we make one of the asynchronous follower synchronous and copy all the change log to the new synchronous follower.
This guarantees that atleast 2 nodes have the recent upto date copy of the data.

![Semi-Replication](https://media.geeksforgeeks.org/wp-content/uploads/20240224115136/Semi-Synchronous-Replication.webp)

In the above figure, we can observe that the replication from the Master replica to Replica 1 is synchronous while the replication to Replica 2 is Asynchronous.
