## PostgreSQL-14 Streaming Replication Cookbook


Here, we will be looking at the step-by-step guide on how to set up _PostgreSQL 14_ streaming replication on _Ubuntu 20.04 LTS_. Like many popular database platforms, PostgreSQL offers its own replication solution that offers data high-availability and is fairly easy to implement.

1. [Database Replication & Types](#1-database-replication--types)
2. [Getting Ready](#2-getting-ready)
    2.1 [Prerequisite](#21-prerequisite)
    2.2 [Configure the primary node](#22-configure-the-primary-node)
    2.3 [Setup Standby Node](#23-setup-standby-node)
3. [How It Works](#3-how-it-works)
    3.1 [Check Database Replication](#31-check-database-replication)
4. [There’s More](#4-theres-more)
    4.1 [Enable Synchronous Replication](#41-enable-synchronous-replication)
    4.2 [Check Synchronous Replication is Done or Not](#42-check-synchronous-replication-is-done-or-not)
    4.3 [Promote Standby Server to New_Primary](#43-promote-standby-server-to-new_primary)
    4.4 [Checking the performance of replication](#44-checking-the-performance-of-replication)
    4.5 [Checking the lag behind among the nodes](#45-checking-the-lag-behind-among-the-nodes)
    4.6 [Stop Replication](#46-stop-replication)
5. [Conclusion](#5-conclusion)

### 1. Database Replication & Types 

Q. What is Database Replication?

Database replication is the **`frequent electronic technique/process copying of data`** from a source database in one computer or server to one or more target databases in another, so that all users share the same level of information. The result is a distributed database in which users can quickly access data relevant to their tasks without interfering with the work of others.

Replication types in PostgreSQL?
1. Logical: replicates data only
2. Streaming: replicates whole database cluster 

Here, we will discuss ‘Streaming Replication’.


### 2. Getting Ready

#### 2.1 Prerequisite

We will be using two nodes with **Ubuntu 20.04 LTS** installed as the operating system to implement our streaming replication cluster. To install **PostgreSQL 14** on Ubuntu please follow the document 

[PostgreSQL: Linux downloads (Ubuntu)](https://www.postgresql.org/download/linux/ubuntu/)

Please note that this tutorial will work on other versions of Ubuntu and other Linux distributions

Primary Node:  172.16.4.215
Standby Node: 172.16.5.155

> _Note:_ Both primary and standby `postgresql.conf` configuration parameters have to be the same. 


#### 2.2 Configure the primary node


Go to the primary node instance.On the primary node switch account to the postgres user. So that it can allow outside connections. 

_Step 1: Configure the **`postgresql.conf`** file_

```bash
sudo vim /etc/postgresql/14/main/postgresql.conf
```

```
listen_addresses = '*' 
wal_level = replica
wal_log_hints = on
max_wal_senders = 3
hot_standby = on
```


##### Definations

**`wal_level`**: Used to _enable PostgreSQL streaming replication_. Possible values are _replica, minimal, and logical_
**`wal_log_hints`**: Required for the **`pg_rewind`** capability, this _helps when the standby server is out of sync with the primary server_
**`max_wal_senders`**: Used to specify the _maximum number of concurrent connections_ that can be established with the standby servers. Must be set to 3 if you are starting with one slave.
**`hot_standby`**: This parameter enables a _read-only connection_ with the slave when it is set to ON.


##### Optional (got from E-Multiskills Database Tutorials)

```
archive_mode: on            # for that we need to create archive folder for log files
archive_commands:
max_wal_senders: 10
wal_keep_size: 500
synchronous_commit = on       # Specifies whether the transaction commit will wait for WAL records to be written to disk before the command returns a “success” indication to the client. The valid values are on, remote_apply, remote_write, local, and off. The default value is on.
```

_Step 2: Restart the postgresql service as a root user_

```bash
sudo systemctl restart postgresql.service
```


_Step 3: Create replication user in primary node_

We will be creating a replication user named replicator in the primary node instance that will be used to facilitate connections between the primary and standby node.  To do this we will be using the createuser utility program. We should now be back on our Linux command-line.

```bash
ubuntu@replication-server1:~$ createuser --replication -P -e replicator
Enter password for new role:
Enter it again: 
```

Enter a password for the new role and remember that password. _The password will be needed at the time of replication_

There are **`many ways`** to create the user, this is just my preferred way of doing it. Another process is running the following command as **`Postgres superuser`**

```sql
CREATE ROLE replicator WITH REPLICATION PASSWORD 'replication' LOGIN;
```

_Step 4: Edit **`pg_hba.conf`** file of the primary node_

Now that our replication user has been created. It is time to configure the user to access our standby node. To do this, we have to edit and set rules in our pg_hba.conf file. 

- Add **`replicator`** user with the replication privilege in the **`pg_hba.conf`** file

```bash
sudo vim /etc/postgresql/14/main/pg_hba.conf
```
```
host	replication 	replicator  	<standby_node_IP> 	md5
```

```
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication 	all                                 	peer
host	replication 	all         	127.0.0.1/32        	scram-sha-256
host	replication 	all         	::1/128             	scram-sha-256
host	replication 	replicator  	172.16.5.155/0      	md5
```

- Restart the postgresql service as a root user 

```
sudo su
systemctl restart postgresql
systemctl reload postgresql.service
```

#### 2.3 Setup Standby Node

_Step 1: Backup standby server data files in another directory_

Now, we will take a backup of the database on the standby node before deleting the data. copy and execute the codes below individually in the standby node instance.

```
sudo su - postgres
sudo systemctl stop postgresql.service
sudo systemctl status postgresql.service
cp -R /var/lib/postgresql/14/main /var/lib/postgresql/14/main_backup
rm -rf /var/lib/postgresql/14/main/*
```

> _Note:_ Check the `/var/lib/postgresql/14/main` directory **ownership**. If the directory ownership is root or any other user except postgres, then you **`can’t start`** the `postgresql.service` later 

_Step 2: Backup data files from primary server_

Now that we have taken a backup of the secondary node and deleted the existing database. We will now need to backup and restore the primary database to the standby node using the pg_basebackup utility tool. Run the following command line in the standby server terminal.

- Run **`pg_basebackup`** from the standby terminal for creating backup from master 

```shell
postgres@replication-server2:~$ pg_basebackup -h 172.16.4.215 -D /var/lib/postgresql/14/main -U replicator -P -v -R -X stream -C -S standby
```

**-h**  – Represents the node that will be backed up. In our case this will be our _primary node_(172.16.4.215)
**-D** – The _data directory_ where the backup will be transferred to.
**-U** – _Connection user_
**-P** – Turns on progress reporting.
**-v** – Enables _verbose mode_
**-R** – Creates a _standby.signal_ file and append connection settings to _postgresql.auto.conf_
**-X** – includes the required write-ahead log files in the backup
**-C** – _enables the creation of a replication slot named_ by the -S option before starting the backup.
**-S** – specifies the _replication slot name_. In the we specify the name ‘standby’


- Provide Password 

After then, it will ask you for the ‘replicator’ user password for completing the backup process. Provide the password. 

### 3. How It Works

#### 3.1 Check Database Replication 

_Step 1: Check from primary node_
We can now head back over to the primary node and check the database replication slot by running:

```shell
sudo su - postgres
psql 
```

```sql
SELECT * FROM pg_replication_slots;
```


If you are following the above instructions, then you should see a replication slot_name = ‘standby’

_Step 2: Start the standby node_

Now we need to head back over to the standby node and start up the services on that database

- Start the postgresql service 

```shell
sudo systemctl start postgresql.service
```
- There should be a connection between our 2 nodes, to confirm this we check the **`pg_stat_wal_receiver`** view by running the following sql command in standby node.

```shell
$ sudo su - postgres
$ psql
```


```sql
SELECT * FROM pg_stat_wal_receiver;
-[ RECORD 1 ]
pid                   | 5859
status                | streaming
receive_start_lsn     | 0/24000000syst
receive_start_tli     | 1
written_lsn           | 0/240002E0
flushed_lsn           | 0/240002E0
received_tli          | 1
last_msg_send_time    | 2022-07-03 10:32:28.352355+00
last_msg_receipt_time | 2022-07-03 10:32:28.355254+00
latest_end_lsn        | 0/240002E0
latest_end_time       | 2022-07-03 08:45:43.96647+00
slot_name             | standby
sender_host           | 172.16.4.215
sender_port           | 5432
conninfo              | user=replicator password=******** channel_binding=prefer dbname=replication host=172.16.4.215 port=5432 fallback_application_name=14/main sslmode=prefer sslcompression=0 sslsni=1 ssl_min_protocol_version=TLSv1.2 gssencmode=prefer krbsrvname=postgres target_session_attrs=any
```

_Step 3: Check postgresql.auto.conf file for sure that everything is OK_

```
cat /var/lib/postgresql/14/main/postgresql.auto.conf
```

```shell
root@replication-server2:/var/lib/postgresql/14/main# cat postgresql.auto.conf
# Do not edit this file manually!
# It will be overwritten by the ALTER SYSTEM command.
primary_conninfo = 'user=replicator password=replication channel_binding=prefer host=172.16.4.215 port=5432 sslmode=prefer sslcompression=0 sslsni=1 ssl_min_protocol_version=TLSv1.2 gssencmode=prefer krbsrvname=postgres target_session_attrs=any'
primary_slot_name = 'standby'
```


Now that we have completed setting up Postgresql 14 streaming replication, it is now time to test it. By default it is an asynchronous streaming replication. 

> _Note:_ PostgreSQL streaming replication is **asynchronous by default**. If the primary server crashes then some transactions that were committed may not have been replicated to the standby server, causing **data loss**. The amount of data loss is proportional to the replication delay at the time of failover.




### 4. There’s More

#### 4.1 Enable Synchronous Replication


Q. Why choose synchronous replication over asynchronous?

In PostgreSQL 14 streaming replication is asynchronous by default. Being asynchronous, there is a high possibility of data loss if the primary node should go offline. Between the primary and secondary node, it usually takes a couple of milliseconds (depending on your network speed) to send the committed data to the secondary node.  To avoid the data loss that occurs in the asynchronous mode, we can put our streaming replication in synchronous mode. In synchronous mode, data is committed to the primary and one or more secondary nodes simultaneously thus lessening the risk of data loss.

To enable synchronous replication, we need to connect to the primary database and execute the command.


**Best Approach(Recommended):**

_Step 1: By default,  `cluster_name = '14/main'` in the `postgresql.conf` configuration file. Change cluster_name parameter in the standby server, so that we can identify the servers individually. Give your desired name to identify the server._

```
cluster_name = 's1'
```


_Step 2: Configure the `postgresql.conf` file in the primary server. Uncomment the  `‘synchronous_standby_names’` parameter and set the standby server name._

```
synchronous_standby_names = 's1'
```

_Step 3: Restart both primary and standby server to take effect_

**Another Approach:**

- _Alter system set as postgres user_

```shell
sudo su - postgres
psql
```

```sql
ALTER SYSTEM SET synchronous_standby_names TO  '*'; 
ALTER SYSTEM
```


> _Note_: synchronous_standby_names to '*' means that all priority of standby server are same and the standby server, which connected to the master server at first, become **SYNC standby**, another server become ASYNC standby as potential server.

- For the changes to take effect we need to reload the postgresql

```shell
sudo su - postgres
psql
```

```sql
SELECT pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)
```


#### 4.2 Check Synchronous Replication is Done or Not

Now execute the command below _in the primary node_, we should see that the database is in sync mode.

```shell
sudo su - postgres
psql
```

```sql
SELECT * FROM pg_stat_replication;
[ RECORD 1 ]
pid          	| 50186
usesysid     	| 21238
usename      	| replicator
application_name | 14/main
client_addr  	| 172.16.5.155
client_hostname  |
client_port  	| 58544
backend_start	| 2022-07-03 14:40:42.498122+06
backend_xmin 	|
state        	| streaming
sent_lsn     	| 0/240001F8
write_lsn    	| 0/240001F8
flush_lsn    	| 0/240001F8
replay_lsn   	| 0/240001F8
write_lag    	|
flush_lag    	|
replay_lag   	|
sync_priority	| 1
sync_state   	| sync
reply_time   	| 2022-07-03 14:41:33.334105+06



SELECT pid,usename,application_name,state,sync_state FROM pg_stat_replication;
pid  |  usename   | application_name |   state   | sync_state  
------+------------+------------------+-----------+------------
1017 | replicator | 14/main          | streaming | sync
(1 row)
```
> _Note:_ Set the application name to in _postgresql.conf_ for synchronous/asynchronous purpose


#### 4.3 Promote Standby Server to New_Primary

Prerequisite: If and only if the primary server is **`down/destroyed`** then follow the below steps.
1. Check the standby. Run the following query in standby.

```sql
SELECT * FROM pg_stat_wal_receiver;
```
_If the query output shows nothing_, then the primary server is down/destroyed or something

2. Run the following line in terminal as postgres user

```bash
CentOS: /usr/pgsql-13/bin/pg_ctl promote -D /var/lib/pgsql/13/data
Ubuntu: /usr/bin/pg_ctlcluster <version> <cluster> <action> [-- <pg_ctl options>]
sudo pg_ctlcluster 14 main promote $PGDATA
```

> _Note:_ After promoting the standby node as new_master, we **can’t** INSERT/UPDATE/DELETE into old_master if we start the postgres service of the old_master.

#### 4.4 Checking the performance of replication

Now, a question may arise that if the streaming replication degrades or increases the performance of the database cluster. To check, we can suggest some tips that you can play on your replication clusters.

> _Note:_ The hardware configurations should be same in the cluster(primary and one or more standbys) 

- Execute several complex queries in all nodes
- Check query execution time(EXPLAIN ANALYZE) in both primary and standby nodes
- Execute the same complex query in a non-replicated node of same hardware config
- If the non-replicated node costs less for a query than you should probably upgrade the hardware components of the replicated cluster. 
- At last, put the results in a google sheet and make a graph to analyze the performance better. 

#### 4.5  Checking the lag behind among the nodes
To monitor how much is lag behind the slots, run this query in the primary node

```sql
SELECT redo_lsn, slot_name,restart_lsn, round((redo_lsn-restart_lsn) / 1024 / 1024 / 1024, 2) AS GB_behind FROM pg_control_checkpoint(), pg_replication_slots;
```

#### 4.6 Stop Replication

Change the postgresql.auto.conf file both from master and standby server.
Remove the standby.signal file from standby server


1. Change the **postgresql.auto.conf** file both from master and standby server.
2. Remove the **standby.signal** file from standby server


### 5. Conclusion

Following this, you should have a working PostgreSQL 14 streaming replication on Ubuntu. To add more nodes you can replicate the steps above.

```sql
DROP A REPLICATION SLOT
select pg_drop_replication_slot(slot_name) from pg_replication_slots where slot_name = 'bottledwater';
```


Ours name is = '14/standby1'
