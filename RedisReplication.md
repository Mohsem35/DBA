Redis is a fast **`in-memory database and cache`** that is built in C and tuned for speed. Redis stands for **`Remote Dictionary Server`** and it is open source 
under the BSD license.

Redis holds its database entirely in the **`memory`**, using the **`disk only for persistence`**

Hence Redis is often referred to as a **`data structure server`**. 

Redis is a **`TCP server`** and supports **`request/response protocol`**. In Redis, a request is accomplished with the following steps −
- The client sends a query to the server, and reads from the socket, usually in a blocking way, for the server response.
- The server processes the command and sends the response back to the client.

### Contents

1. [Redis Advantages](#redis-advantages)
2. [Redis Commands](#redis-commands)
3. [Redis Data Types](#redis-data-types)
4. [Redis Publish Subscribe](#redis-publish-subscribe)
5. [Redis Backup Restore](#redis-backup-restore)
6. [Redis Replication](#replication-setup)
7. [Redis Security](#redis-security)


Q. Redis is a in memory database, so after server restart why the redis databases doesn't wipe?

1. `Snapshots`: Redis can periodically save snapshots of the **`in-memory data to disk`**. These snapshots are essentially **`point-in-time copies`** of the dataset, and they are saved as binary files on disk. By default, Redis saves snapshots every **`15 minutes`** if at least one key has changed.

For example, this configuration will make Redis automatically dump the dataset to disk every 60 seconds if at least 1000 keys changed:
```
save 60 1000
```
This strategy is known as **`snapshotting`**.
  
3. `Append-Only Files (AOF)`: In addition to snapshots, Redis can use an Append-Only File (AOF) for persistence. The AOF **`logs every write operation`** to the database, so it contains a complete record of all changes made to the data. By replaying the AOF log, Redis can reconstruct the dataset from scratch. This log is written in an append-only manner, which means it is **`continuously updated`** with new write operations.

You can turn on the AOF in your configuration file:

```
appendonly yes
```

3. `Combination of Snapshots and AOF`: Redis often uses a combination of both snapshots and AOF for persistence. When Redis restarts, it can use the latest snapshot to load the dataset into memory and then replay the AOF log to bring it up to the current state.

So, even if your PC is restarted, Redis can recover its data from these snapshots and/or the AOF log. This is why Redis is often described as having a **`hybrid`** in-memory and on-disk storage model that provides durability and persistence, even in the face of system crashes or restarts.


## Redis Installation

```shell
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update
sudo apt-get install redis-server
sudo apt-get install redis
```
```shell
sudo vim /etc/redis/redis.conf
```

```
. . .

# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised systemd
. . .
```

```shell
sudo systemctl enable redis-server --now
sudo systemctl restart redis.service
sudo systemctl status redis.service
```

### Redis Advantages

**1. `Exceptionally fast`** − Redis is very fast and can perform about _110000 SETs_/second, about _81000 GETs_/second.

**2. `Supports rich data types`** − Redis natively supports most of the datatypes that developers already know such as `list`, `set`, `sorted set`, `strings` and `hashes`, . This makes it easy to solve a variety of problems as we know which problem can be handled better by which data type.

**3. `Operations are atomic`** − All Redis operations are atomic, which ensures that if _two clients concurrently access_, Redis server will receive the updated value.

**4. `Multi-utility tool`** − Redis is a multi-utility tool and can be used in a number of use cases such as **`caching`**, **`messaging-queues`** (Redis natively supports Publish/Subscribe), **`any short-lived data in your application`**, such as web application sessions, web page hit counts, etc.


[Install Redis on Linux](https://redis.io/docs/getting-started/installation/install-redis-on-linux/)


## Redis Commands

#### Start redis

```shell
sudo systemctl start redis-server.service redis.service
```

#### Redis version

```shell
redis-server -v
Redis server v=7.2.1 sha=00000000:0 malloc=jemalloc-5.3.0 bits=64 build=95712a67f5005c28
```


#### Check if Redis is working

```shell
redis-cli
redis 127.0.0.1:6379> ping 
PONG
```

#### Redis server status

```shell
sudo systemctl status redis-server.service
```


#### Connect with remote Redis server

```
redis-cli -h 10.0.0.1 -p 6379 -a password
```

#### Redis data directory

```shell
127.0.0.1:6379> CONFIG get dir  
1) "dir" 
2) "/var/lib/redis" 
```

`/etc/redis` is the redis _configuration directory_, where the file name is **`redis.conf`**


#### List all databases in Redis

```
redis-cli INFO keyspace
```

#### Delete/Empty a Redis database

```shell
redis-cli -n 8 flushdb              # deletes specific db8
```
```shell
redis-cli flushall                  # deletes all dbs
```

#### Redis user list

```shell
redis-cli
127.0.0.1:6379> ACL USERS
1) "default"
127.0.0.1:6379> ACL WHOAMI
"default"
```


#### Redis configuration settings _(GET)_

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



#### Connect redis from GUI tools

```
vim /etc/redis/redis.conf
```
```
bind 127.0.0.1 <own_lan_ip> ::1
protected-mode no
```
```
sudo systemctl stop redis-server.service
systemctl daemon-reload
sudo systemctl start redis-server.service
```


## Redis Security

Redis database can be secured, such that any client making a connection needs to authenticate before executing a command. To secure Redis, you need to set the password in the config file. Following example explains how a client authenticates itself to Redis server and checks whether the server is running or not.


#### Set password

```
127.0.0.1:6379> CONFIG get requirepass 
1) "requirepass" 
2) ""
```
By default, this property is blank, which means no password is set for this instance. You can change this property by executing the following command.

```shell
127.0.0.1:6379> CONFIG set requirepass "password" 
OK 
127.0.0.1:6379> CONFIG get requirepass 
1) "requirepass" 
2) "password" 
```
After setting the password, if any client _runs the command without authentication_, then _(error) NOAUTH Authentication required_. error will return. Hence, the client needs to use `AUTH` command to **`authenticate himself`**.

```
redis 127.0.0.1:6379> AUTH "password" 
OK 
redis 127.0.0.1:6379> PING 
PONG
```


## Redis Data Types


#### Redis - Keys

```
redis 127.0.0.1:6379> SET key value
OK 
redis 127.0.0.1:6379> DEL key
(integer) 1
```

In the above example, `DEL` is the command. If the key is deleted, then the output of the command will be (integer) 1, otherwise it will be (integer) 0.

#### Redis - Strings

```shell
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

## Redis Publish Subscribe 

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


#### Redis - Server

Redis server commands are basically used to manage Redis server.

Following example explains how we can **`get all statistics and information about the server`**

```
redis 127.0.0.1:6379> INFO  
```

## Redis Backup Restore

Step 1: Redis `SAVE` command is used to create a **`backup`** of the current Redis database.

```
127.0.0.1:6379> SAVE  
OK
```

This command will create a **`dump.rdb`** file in your Redis directory.

The SAVE commands performs a _synchronous save of the dataset producing a point in time snapshot_ of all the data inside the Redis instance, in the form of an RDB file. You almost **`never`** want to call SAVE in **`production`** environments where it will **`block`** all the other clients. Instead usually **`BGSAVE`** is used. This command will start the backup process and run this in the **`background`**.

```
127.0.0.1:6379> BGSAVE  
Background saving started
```

step 2: To **`restore`** Redis data, move Redis backup file (dump.rdb) into your reomte Redis server directory

```
scp /var/lib/redis/dump.rdb hostname@ip:/home/ubuntu
```

step 3: Stop the restore serve 

```
sudo systemctl stop redis-server.service
```

step 4: Find out the restore server redis data directory and move the `dump.rdb` dump to that directory

```
127.0.0.1:6379> CONFIG get dir  
1) "dir" 
2) "/var/lib/redis" 
```
```
mv /home/ubuntu/dump.rdb /redis/data/directory 
```

step 5: Restart daemon and start redis server 

```shell
sudo systemctl start redis-server.service
```


## Redis configuration parameters that we should know

```shell
sudo vim /etc/redis/redis.conf
```

```shell
maxclients 10000                           # Max number of connected clients at the same time
supervised systemd
bind 127.0.0.1 172.16.6.151 ::1            # Listens on specific IPv4 addresses with IPv6
protected-mode no                          # Clients from other hosts to connect to Redis
logfile /var/log/redis/redis-server.log  
dbfilename dump.rdb                        # Dump file name
replica-read-only yes
```

## Redis Partitioning:

Redis Partitioning: is the process of splitting your data into multiple Redis instances, so that every instance will only contain a subset of your keys.

- **`Range`** partitioning is accomplished by mapping ranges of objects into specific Redis instances. Suppose in our example, the users from ID 0 to ID 10000 will go into instance R0, while the users from **`ID 10001 to ID 20000`** will go into instance R1 and so forth.
  
- **`Hash`** Partitioning is a **`hash function`** (eg. modulus function) is used to convert the **`key into a number`** and then the data is stored in different-different Redis instances.


## Redis Data Replication

### Features of Redis Data Replication

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

This system works using three main mechanisms:

- When a master and a replica instances are **`well-connected`**, the master keeps the replica updated by sending a **`stream of commands to the replica`** to replicate the effects on the dataset happening in the master side due to: client writes, keys expired or evicted, any other action changing the master dataset.

- When the link between the master and the **`replica breaks`**, for network issues or because a timeout is sensed in the master or the replica, the replica **`reconnects`** and attempts to proceed with a **`partial resynchronization`**: it means that it will try to just obtain the part of the stream of commands it missed during the disconnection.

- When a **`partial resynchronization is not possible`**, the replica will ask for a **`full resynchronization`**. This will involve a more complex process in which the master needs to create a snapshot of all its data, send it to the replica, and then continue sending the stream of commands as the dataset changes.


### Replication Setup
- Replicas support a **`read-only`** mode that is enabled by **`default`**

redis01 (Master): 172.16.6.18

redis02 (Slave): 172.16.6.57



#### Configuring Redis Master Server(172.16.6.18)

```bash
sudo vim /etc/redis/redis.conf
```

By default, Redis is configured to **`listen and accept`** connections on the **`loopback interface`** using the **`bind`** directive.


- To enable communication between replicas, the _master_ should be configured to _listen IPv4 loopback address and its OWN LAN IP address 172.16.6.18_
- For _secure communications_, we can protect the master using the **`requirepass`** directive, so that the clients/replicas request an authentication password before running any commands.
- We then configure _Redis to interact with systemd_. To do this, the **`supervised`** parameter needs to be added



```
bind 127.0.0.1 172.16.6.18 ::1
protected-mode no
requirepass DLT@DevOps44                # use the AUTH password which we use in redis-cli
supervised systemd
```

Once this is done, save the file and close it. Restart the Redis service for the changes to take effect.
```
systemctl daemon-reload
systemctl restart redis
```

#### Configuring Redis' Replica servers(172.16.6.57)

On the replica servers, we need to configure in the **`same way`** we did for the master. Add the respective IPv4 address in the bind directive on both the replica servers.

On 1st replica server

```
bind 127.0.0.1 172.16.6.57 ::1
```
Now, we set up our Redis instance as a replica

In order to configure the Redis instance as a **`replica`**, we can use the **`replicaof`** parameter and set the master node's IP and port to identify the master and enable communication.
- authentication on the replica nodes will use the **`masterauth`** parameter

```
replicaof 172.16.6.18 6379
masterauth DLT@DevOps44
requirepass passwordslave
```
```
sudo systemctl daemon-reload
sudo systemctl restart redis
```

#### Verifying master-replica status

On the master node run the following commands

![Screenshot from 2023-09-10 14-55-04](https://github.com/Mohsem35/DBA/assets/58659448/392190f0-a439-4c9e-962e-02f05fc95736)

Check the replication status on the slaves as well

![Screenshot from 2023-09-10 14-59-37](https://github.com/Mohsem35/DBA/assets/58659448/a1ae606e-5fb4-48e1-b89e-3ae3ff89f055)

### Redis Replication Failover(replica to new_master)

If master node is broken and/or down, replica, the current slave, needs to be made to the **`new master`**. First let's check for the current replication setup on replica: 

```
redis-cli info
```

Let's tell replica to become a new_master server: 
```
redis-cli
127.0.0.1:6379> slaveof no one
OK 
```
Verify with info again.


After a while, when **`old_master is fixed`**, new_master can be connected as slave to old_master to sync the new data. Run the following commands to the new_master server

```
redis-cli
127.0.0.1:6379> slaveof 172.16.6.18 6379
```
Verify with info again.

[Redis Failover Documentation](https://redis.io/commands/slaveof/)

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

[Another approach for Redis Replication](https://developer.redis.com/operate/redis-at-scale/high-availability/exercise-1)


[Redis Replication](https://medium.com/dlt-labs-publication/how-to-setup-redis-replication-c9cc89ba6c03#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjgzOGMwNmM2MjA0NmMyZDk0OGFmZmUxMzdkZDUzMTAxMjlmNGQ1ZDEiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJhenAiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJhdWQiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJzdWIiOiIxMDE2MjQ3Mzk5MzU4MTQ2MjkyMTMiLCJoZCI6ImNlbGxvc2NvLnBlIiwiZW1haWwiOiJtYWx5Lm1vaHNlbUBjZWxsb3Njby5wZSIsImVtYWlsX3ZlcmlmaWVkIjp0cnVlLCJuYmYiOjE2OTQwNjk3ODIsIm5hbWUiOiJNYWx5IE1vaHNlbSBBaG1lZCIsInBpY3R1cmUiOiJodHRwczovL2xoMy5nb29nbGV1c2VyY29udGVudC5jb20vYS9BQWNIVHRkcUpPZ0M1dnBRa2M0OW1tUDN6c3FSTFZBNTJ1UW5HeERNeW1OU3ZQQTdJZz1zOTYtYyIsImdpdmVuX25hbWUiOiJNYWx5IE1vaHNlbSIsImZhbWlseV9uYW1lIjoiQWhtZWQiLCJsb2NhbGUiOiJlbiIsImlhdCI6MTY5NDA3MDA4MiwiZXhwIjoxNjk0MDczNjgyLCJqdGkiOiJmNzlhODY1ZjgxMzRhYjU3MWRlNDRhOTZlODliYmQ0YmFhNTgwOGNhIn0.MICVLgxNyUSLXD8tuygg_MmDgj83LGIxq9LU-J0d2ubV_TE9BMT8PJEKPMFlle9ZT2yYl2ngxHdVD3-sz4XW1l6jAva15rk3yNwtDpiLJJ-3JFpvmTW9Zxeoy_0f8nwsgTrk45ex1RXgjpskujh5QCd4UEiD4_wMsb3GZl82zck3RLI_-d5NHWyOmIfhUgNlnXsbEmTH2EAqRisjaB2KRnAHCcxmyHaWCqJCpemj-yRNPndApFDkQk_gKkE74SNrUlUpXA0ALVcW-I-fnEX3vY9yZYB6uPJ5B3Gmv4FTjdatfwuiqr5skHjAWQnJ3iCz7Spzv1kWgUdbz5jccQdH8A)
