Redis is a fast **`in-memory database and cache`** that is built in C and tuned for speed. Redis stands for **`Remote Dictionary Server`** and it is open source 
under the BSD license.

Hence Redis is often referred to as a **`data structure server`**. 

## Features of Redis Data Replication

- **`Asynchronous replication`** is used by Redis, with asynchronous replica-to-master acknowledgments of data processed.
- **`Multiple replicas`** are possible for a master.
- Connections from other replicas can be accepted by replicas. Aside from linking several replicas to the same master, replicas can also be linked to each other
  in a cascading pattern. Since Redis 4.0, the master sends the identical Redis data replication stream to all sub-replicas.
- On the master side, **`Redis data replication is non-blocking`**. When one or more replicas execute the initial synchronization or partial resynchronization,
  the **`master`** will continue to **`handle queries`**.
- Redis Data Replication can be used for scalabilities, such as having **`several replicas for read-only queries`** (slow O(N) operations can be offloaded to replicas)
  or just to improve data safety and availability.
- You can use Redis data replication to avoid the cost of the master writing the entire dataset to disc: a standard strategy involves configuring your master
  redis.conf to avoid persisting at all, then connecting a replica configured to save periodically or with AOF enabled. This setup, however, must be treated with
  caution because a restarted master will start with an empty dataset, and if the replica attempts to sync with it, the replica will be emptied as well.


## Redis Data Replication Process


### How Redis Data Replication Works?

Every Redis master has a Redis data **`replication ID`**, which is a long pseudo-random string that identifies a certain dataset tale.

Every byte of the replication stream that is produced and transferred to replicas **`increments an offset`**, which is used to update the state of the replicas 
with the **`new modifications`** changing the dataset. 

Even if no replica is connected, the Redis data replication offset is incremented, thus basically every pair of:

```
Replication ID, Offset
```

When **`replicas`** connect to masters, they send their **`previous master replication ID and the offsets`** they’ve processed so far using the **`PSYNC`** 
command

This allows the master to deliver **`only the incremental parts`** that are required.

If there is an insufficient backlog in the master buffers, or if the replica is referring to a history (replication ID) that is no longer valid, a full 
resynchronization occurs: the replica will receive a fresh copy of the dataset.


![ezgif com-webp-to-jpg](https://github.com/Mohsem35/DBA/assets/58659448/870b21e6-8fdc-44e8-9c8f-58e46b0728e6)



### How Redis Data Replication Works?

A **`replication ID`** identifies a certain **`data set’s history`**. A unique replication ID is generated for this instance every time it starts over as a 
master or a replica is promoted to master.

After the **`handshake`**, replicas connected to a master will **`inherit its replication ID`**. So two instances with the **`same ID`** are linked since they each have the same 
data but at a different time.

For a given history (replication ID) that has the most current data set, it is the offset that acts as a logical time to grasp.

For example, if two instances A and B both have the same replication ID, but one has offset **`1000`** and the other has offset **`1023`**, the **`first is missing some 
commands`** applied to the data set.


