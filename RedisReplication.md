Redis is a fast **`in-memory database and cache`** that is built in C and tuned for speed. Redis stands for **`Remote Dictionary Server`** and it is open source 
under the BSD license.

Redis holds its database entirely in the **`memory`**, using the **`disk only for persistence`**

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

## Redis Advantages

**`Exceptionally fast`** − Redis is very fast and can perform about _110000 SETs_/second, about _81000 GETs_/second.

**`Supports rich data types`** − Redis natively supports most of the datatypes that developers already know such as `list`, `set`, `sorted set`, `strings` and `hashes`, . This makes it easy to solve a variety of problems as we know which problem can be handled better by which data type.

**`Operations are atomic`** − All Redis operations are atomic, which ensures that if _two clients concurrently access_, Redis server will receive the updated value.

**`Multi-utility tool`** − Redis is a multi-utility tool and can be used in a number of use cases such as **`caching`**, **`messaging-queues`** (Redis natively supports Publish/Subscribe), **`any short-lived data in your application`**, such as web application sessions, web page hit counts, etc.


## Redis Commands

- Start redis

```
redis-server
```

- Check If Redis is Working

```
redis-cli

redis 127.0.0.1:6379> ping 
PONG
```
- Redis configuration settings _(GET)_

```
redis 127.0.0.1:6379> CONFIG GET <config_setting_name>
```
```
redis 127.0.0.1:6379> CONFIG GET loglevel  
1) "loglevel" 
2) "notice"
```
- Redis configuration settings _(SET)_

```
redis 127.0.0.1:6379> CONFIG SET <config_setting_name> <new_config_value>
```
```
redis 127.0.0.1:6379> CONFIG SET loglevel "notice" 
OK 
```

#### Redis - Keys

```
redis 127.0.0.1:6379> SET key value
OK 
redis 127.0.0.1:6379> DEL key
(integer) 1
```

In the above example, `DEL` is the command. If the key is deleted, then the output of the command will be (integer) 1, otherwise it will be (integer) 0.

#### Redis - Strings

```
redis 127.0.0.1:6379> SET tutorialspoint redis 
OK 
redis 127.0.0.1:6379> GET tutorialspoint 
"redis" 
```
In the above example, SET and GET are the commands, while tutorialspoint is the key.


#### Redis - Hashes

Redis Hashes are **`maps between`** the string fields and the string values. Hence, they are the perfect data type to **`represent objects`**. In Redis, every hash can store up to more than 4 billion field-value pairs.

```
redis 127.0.0.1:6379> HMSET key field value [field value ...]
```

```
redis 127.0.0.1:6379> HMSET tutorialspoint name "redis tutorial" 
description "redis basic commands for caching" likes 20 visitors 23000 
OK 
redis 127.0.0.1:6379> HGETALL tutorialspoint  
1) "name" 
2) "redis tutorial" 
3) "description" 
4) "redis basic commands for caching" 
5) "likes" 
6) "20" 
7) "visitors" 
8) "23000"
```
In the above example, we have set Redis tutorials detail (name, description, likes, visitors) in hash named ‘tutorialspoint’.


#### Redis - Lists

Redis Lists are simply lists of strings, **`sorted by insertion order`**. You can add elements in Redis lists in the head or the tail of the list.

```
redis 127.0.0.1:6379> LPUSH tutorials redis 
(integer) 1 
redis 127.0.0.1:6379> LPUSH tutorials mongodb 
(integer) 2 
redis 127.0.0.1:6379> LPUSH tutorials mysql 
(integer) 3 
redis 127.0.0.1:6379> LRANGE tutorials 0 10  
1) "mysql" 
2) "mongodb" 
3) "redis"
```


#### Redis - Sets

Redis Sets are an **`unordered collection of unique strings`**. Unique means sets does not allow repetition of data in a key.

In Redis set add, remove, and test for the existence of members in O(1) 

```
redis 127.0.0.1:6379> SADD tutorials redis 
(integer) 1 
redis 127.0.0.1:6379> SADD tutorials mongodb 
(integer) 1 
redis 127.0.0.1:6379> SADD tutorials mysql 
(integer) 1 
redis 127.0.0.1:6379> SADD tutorials mysql 
(integer) 0 
redis 127.0.0.1:6379> SMEMBERS tutorials  
1) "mysql" 
2) "mongodb" 
3) "redis"
```
In the above example, three values are inserted in Redis set named ‘tutorials’ by the command SADD.


#### Redis - Sorted Sets


Redis Sorted Sets are **`similar`** to Redis Sets with the unique feature of values stored in a set. The difference is, every member of a **`Sorted Set is associated with a score`**, that is used in order to take the sorted set ordered, from the smallest to the greatest score.

In Redis sorted set, add, remove, and test for the existence of members in O(1) 

```
redis 127.0.0.1:6379> ZADD key score member [score member ...]
```

```
redis 127.0.0.1:6379> ZADD tutorials 1 redis 
(integer) 1 
redis 127.0.0.1:6379> ZADD tutorials 2 mongodb 
(integer) 1 
redis 127.0.0.1:6379> ZADD tutorials 3 mysql 
(integer) 1 
redis 127.0.0.1:6379> ZADD tutorials 3 mysql 
(integer) 0 
redis 127.0.0.1:6379> ZADD tutorials 4 mysql 
(integer) 0 
redis 127.0.0.1:6379> ZRANGE tutorials 0 10 WITHSCORES  
1) "redis" 
2) "1" 
3) "mongodb" 
4) "2" 
5) "mysql" 
6) "4" 
```

In the above example, three values are inserted with its score in Redis sorted set named ‘tutorials’ by the command ZADD.

#### Redis - HyperLogLog


Redis HyperLogLog is an **`algorithm`** that uses randomization in order to provide an **`approximation`** of the number of **`unique elements in a set`** using just a constant, and small amount of memory.

HyperLogLog provides a very good approximation of the cardinality of a set even using a very small amount of memory around 12 kbytes per key with a standard error of 0.81%. There is no limit to the number of items you can count, unless you approach 2^64 items.

```
redis 127.0.0.1:6379> PFADD tutorials "redis"  
1) (integer) 1  
redis 127.0.0.1:6379> PFADD tutorials "mongodb"  
1) (integer) 1  
redis 127.0.0.1:6379> PFADD tutorials "mysql"  
1) (integer) 1  
redis 127.0.0.1:6379> PFCOUNT tutorials  
(integer) 3 
```

#### Redis - Publish Subscribe 

Redis Pub/Sub implements the **`messaging system`** where the senders (in redis terminology called **`publishers`**) sends the messages while the receivers (**`subscribers`**) receive them. The link by which the messages are transferred is called **`channel`**.

In Redis, a client can subscribe any number of channels.

In the following example, one client subscribes a channel named ‘redisChat’.
```
redis 127.0.0.1:6379> SUBSCRIBE redisChat  
Reading messages... (press Ctrl-C to quit) 
1) "subscribe" 
2) "redisChat" 
3) (integer) 1 
```

Now, two clients are publishing the messages on the same channel named ‘redisChat’ and the above subscribed client is receiving messages.

```
redis 127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"  
(integer) 1  
redis 127.0.0.1:6379> PUBLISH redisChat "Learn redis by tutorials point"  
(integer) 1   
1) "message" 
2) "redisChat" 
3) "Redis is a great caching technique" 
1) "message" 
2) "redisChat" 
3) "Learn redis by tutorials point" 
```

#### Redis - Transactions


Redis transactions allow the **`execution`** of a _group of commands in a single step_. Following are the two properties of Transactions.

- All commands in a transaction are sequentially executed as a single isolated operation. It is not possible that a request issued by another client is served in the middle of the execution of a Redis transaction.
- Redis transaction is also atomic. Atomic means **`either all of the commands or none are processed`**


Redis transaction is initiated by command `MULTI` and then you need to _pass a list of commands that should be executed in the transaction_, after which the entire transaction is executed by `EXEC` command.

```
redis 127.0.0.1:6379> MULTI 
OK 
List of commands here 
redis 127.0.0.1:6379> EXEC
```
```
redis 127.0.0.1:6379> MULTI 
OK 
redis 127.0.0.1:6379> SET tutorial redis 
QUEUED 
redis 127.0.0.1:6379> GET tutorial 
QUEUED 
redis 127.0.0.1:6379> INCR visitors 
QUEUED 
redis 127.0.0.1:6379> EXEC  
1) OK 
2) "redis" 
3) (integer) 1 
```


#### Redis - Scripting

Redis scripting is used to evaluate scripts using the Lua interpreter. It is built into Redis starting from version 2.6.0. The command used for scripting is `EVAL` command.

Following is the basic syntax of EVAL command.

```
redis 127.0.0.1:6379> EVAL script numkeys key [key ...] arg [arg ...]
```
```
redis 127.0.0.1:6379> EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 
key2 first second  
1) "key1" 
2) "key2" 
3) "first" 
4) "second"
```

```
# understanding the EVAL with values
"return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" = script
2 = numkeys
key1, key2 = key
first, second = arg
```


#### Redis - Connections

Redis connection commands are basically used to manage client connections with Redis server.

Following example explains how a client authenticates itself to Redis server and checks whether the server is running or not.

```
redis 127.0.0.1:6379> AUTH "password" 
OK 
redis 127.0.0.1:6379> PING 
PONG
```

#### Redis - Server

Redis server commands are basically used to manage Redis server.

Following example explains how we can **`get all statistics and information about the server`**

```
redis 127.0.0.1:6379> INFO  
```

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



### What is Redis Data Replication ID?

A **`replication ID`** identifies a certain **`data set’s history`**. A unique replication ID is generated for this instance every time it starts over as a 
master or a replica is promoted to master.

After the **`handshake`**, replicas connected to a master will **`inherit its replication ID`**. So two instances with the **`same ID`** are linked since they each have the same 
data but at a different time.

For a given history (replication ID) that has the most current data set, it is the offset that acts as a logical time to grasp.

For example, if two instances A and B both have the same replication ID, but one has offset **`1000`** and the other has offset **`1023`**, the **`first is missing some 
commands`** applied to the data set.


### INFO and ROLE Command

There are two Redis commands that provide a wealth of information about the master and replica instances’ current replication parameters.

**`INFO`**: Only **`information relevant`** to the replication is presented when the command is run with the replication option as INFO replication.

**`ROLE`**: is a more computer-friendly command that displays the **`replication status of masters and replicas`**, as well as replication offsets, a list of linked replicas, and so on.


