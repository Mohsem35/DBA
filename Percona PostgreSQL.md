[Percona Distribution for PostgreSQL 16 Documentation](https://docs.percona.com/postgresql/16/index.html#)


Percona Distribution for PostgreSQL is a collection of tools to assist you in managing your PostgreSQL database system: it installs PostgreSQL and complements it by a selection of extensions that enable solving essential practical tasks efficiently:

[HAProxy](http://www.haproxy.org/) - _a high-availability and load-balancing solution_

[Patroni](https://patroni.readthedocs.io/en/latest/) _is an HA (High Availability) solution for PostgreSQL._

[pgAudit](https://www.pgaudit.org/) _provides detailed session or object audit logging via the standard PostgreSQL logging facility_

[pgAudit set_user](https://github.com/pgaudit/set_user) - _The `set_user` part of `pgAudit` extension provides an additional layer of logging and control when unprivileged users must escalate themselves to superuser or object owner roles in order to perform needed maintenance tasks._

[pgBackRest](https://pgbackrest.org/) _is a backup and restore solution for PostgreSQL_

[pgBadger](https://github.com/darold/pgbadger) - _a fast PostgreSQL Log Analyzer._

[PgBouncer](https://www.pgbouncer.org/) - _a lightweight connection pooler for PostgreSQL_

pg_gather - _an SQL script to assess the health of PostgreSQL cluster by gathering performance and configuration data from PostgreSQL databases._

[pgpool2](https://www.pgpool.net/mediawiki/index.php/Main_Page) - _a middleware between PostgreSQL server and client for high availability, connection pooling and load balancing._

[pg_repack](https://github.com/reorg/pg_repack) _rebuilds PostgreSQL database objects_

pg_stat_monitor _collects and aggregates statistics for PostgreSQL and provides histogram information._

PostGIS _allows storing and manipulating spacial data in PostgreSQL._

wal2json - _a PostgreSQL logical decoding JSON output plugin._

A collection of additional PostgreSQL contrib extensions


### Patroni 

#### etcd


**`Etcd`** is an **open-source distributed key-value store** that is designed for 
- _configuration management_ 
- _service discovery_
- _maintaining coordination_ across a cluster of machines. 

It was developed by the CoreOS team and is now maintained by the Cloud Native Computing Foundation (CNCF). Etcd is **widely used in distributed systems** and container orchestration platforms like **Kubernetes**


**`etcd`** helps Patroni **nodes to communicate** and **elect a leader among them**. The leader is responsible for managing the primary PostgreSQL node and **making decisions about failovers** and other cluster operations.

When using etcd with Patroni, etcd stores 
- _information about the state of each PostgreSQL node_
- _leader election details_
- _other configuration parameters_

This allows Patroni to achieve consensus among the nodes, making it possible to ensure that only one node acts as the primary at a given time.


Node1: 172.16.7.90

```shell
# node1
sudo apt-get install etcd
cat /etc/default/etcd

ETCD_NAME="node1"
ETCD_INITIAL_CLUSTER="node1=http://192.168.1.11:2380"
ETCD_INITIAL_CLUSTER_TOKEN="devops_token"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.11:2380"
ETCD_DATA_DIR="/var/lib/etcd/postgresql"
ETCD_LISTEN PEER_URLS="http://192.168.1.11:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.1.11:2379,http://localhost:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.11:2379"

sudo systemctl restart etcd
sudo etcdctl member list

# node2
sudo apt-get install etcd
cat /etc/default/etcd

ETCD_NAME="node2"
ETCD_INITIAL_CLUSTER="node1=http://192.168.1.11:2380,node2=http://192.168.1.12:2380"
ETCD_INITIAL_CLUSTER_TOKEN="devops_token"
ETCD_INITIAL_CLUSTER_STATE="existing"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.12:2380"
ETCD_DATA_DIR="/var/lib/etcd/postgresql"
ETCD_LISTEN_PEER_URLS="http://192.168.1.12:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.1.12:2379,http://localhost:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.12:2379"
$ sudo systemctl restart etcd

# node 1
sudo etcdctl member add node2 http://192.168.1.12:2380

# output
Added member named node2 with ID c06ed38e1cdab698 to cluster

ETCD_NAME="node2"
ETCD_INITIAL_CLUSTER="node2=http://172.16.7.91:2380,node1=http://172.16.7.90:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"

# node3
sudo apt-get install etcd
cat /etc/default/etcd

ETCD_NAME="node3"
ETCD_INITIAL_CLUSTER="node1=http://192.168.1.11:2380,node2=http://192.168.1.12:2380,node3=http://192.168.1.13:2380"
ETCD_INITIAL_CLUSTER_TOKEN="devops_token"
ETCD_INITIAL_CLUSTER_STATE="existing"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.13:2380"
ETCD_DATA_DIR="/var/lib/etcd/postgresql"
ETCD_LISTEN PEER_URLS="http://192.168.1.13:2380" 
ETCD_LISTEN_CLIENT_URLS="http://192.168.1.13:2379,http://localhost:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.13:2379"

# node 1
sudo systemctl restart etcd
sudo etcdctl member add node3 http://192.168.1.13:2380
```

Now check the members list from node1
```shell
sudo etcdctl member list
```

### Watchdog

In the context of Patroni, the term **`watchdog`** refers to a mechanism used for **monitoring the health and state of PostgreSQL nodes** within a high-availability cluster. The watchdog component in Patroni plays a crucial role in detecting failures, initiating failovers, and ensuring the overall availability of the PostgreSQL database.

**`Softdog`** is a software version of Watchdog.


```shell
# node1, node2, node 3
sudo sh -c 'echo "softdog" >> /etc/modules'
sudo sh -c 'echo "KERNEL==\"watchdog\", OWNER=\"postgres\", GROUP=\"postgres\"" >> /etc/udev/rules.d/61-watchdog.rules'

# Remove "blacklist softdog" from strain files
grep blacklist /lib/modprobe.d/* /etc/modprobe.d/* |grep softdog /lib/modprobe.d/blacklist_linux_5.4.0-72-generic.conf:blacklist softdog
```

### PostgreSQL

Run the following lines at node1, node2 and node3

```shell
# From Percona Repository
sudo apt-get update -y; sudo apt-get install -y wget gnupg2 lsb-release curl 
wget https://repo.percona.com/apt/percona-release_latest.generic_all.deb 
sudo dpkg -i percona-release_latest.generic_all.deb
sudo apt-get update
sudo percona-release setup ppg-12
sudo apt-get install percona-postgresql-12 etcd patroni

# For this test, leave it to Patroni to do it all
sudo systemctl stop postgresql
sudo rm -fr /var/lib/postgresql/12/main
```

### Patroni 

```shell
# node 1
# install from the percona repository as well
sudo apt-get install patroni

# customize the main configuration file
cat /etc/patroni/config.yml
sudo rm -rf /var/lib/postgresql/12/*

# restart the service
sudo systemctl restart patroni

# remove everything from data directory
sudo su
rm -rf /var/lib/postgresql/12/*
systemctl start patroni

# check status of the cluster
sudo patronictl -c /etc/patroni/config.yml list
```
have to do the previous steps in node 2 and node 3

```shell
# node2 and node3
# remove everything from data directory
sudo su
rm -rf /var/lib/postgresql/12/*
systemctl start patroni
ps aux | grep postgres

# node 1
# check status of the cluster
sudo patronictl -c /etc/patroni/config.yml list
```

> some confusion at 13:12 seconds of the video
> 
### HAproxy

I am going to install HAproxy in another node like my personal computer

- We will create 2 groups. `primary` and `secondary`
- Write --> 5000
- Reads --> 5001

```shell
sudo apt-get install haproxy
sudo cat /etc/haproxy/haproxy.cfg

fernando@fernando-MacBookPro:~$ more /etc/haproxy/haproxy.cfg
global
    maxconn 100
defaults
    Log global
    mode tcp
    retries 2
    timeout client 30m
    timeout connect 4s
    timeout server 30m
    timeout check 5s

listen stats
    mode http
    bind *:7000
    stats enable
    stats uri /

listen primary
    bind *:5000
    option httpchk OPTIONS /master
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server node1 node1:5432 maxconn 100 check port 8008
    server node2 node2:5432 maxconn 100 check port 8008
    server node3 node3:5432 maxconn 100 check port 8008

listen standbys
    balance roundrobin
    bind *:5001
    option httpchk OPTIONS /replica
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server node1 node1:5432 maxconn 100 check port 8008
    server node2 node2:5432 maxconn 100 check port 8008
    server node3 node3:5432 maxconn 100 check port 8008
```
```shell
# store password in a file
$ echo "localhost:5000:postgres:postgres:vagrant" > ~/.pgpass
$ echo "localhost:5001:postgres:postgres:vagrant" >> ~/.pgpass
$ chmod 0600 ~/.pgpass

# run the following commands where the HAproxy package is installed
# test reads
$ psql -Upostgres -hlocalhost -p5001 -t -c "select inet_server_addr()"

# test writes
psql -Upostgres -hlocalhost -p5000 -t -c "select inet_server_addr()"
```

### Workload

create a database with table in node for testing purpose

```shell
# run the following commands where the HAproxy package is installed
sudo apt-get install python3-psycopg2
curl -LO https://raw.githubusercontent.com/jobinau/pgscripts/main/patroni/HAtester.py
chmod +x HAtester.py

# Writes
$ ./HAtester.py 5000

Reads
$ ./HAtester.py 5001 
```

### Testing

Postgres is managed by Patroni here

1. Loss of network communication
  - Unplug network cable from one node
      - Frist a replica
      - Then the primary
  - Unplug network cable from one replica and the primary
    - split brain?  

```shell
sudo su
watch patronictl -c /etc/patroni/config.yml lsit
```
   
2. Power outrage
   - Unplug power cable from the primary
3. SEGFAULT
   - Simulating OOM/creash with "kill -9"
  
4. CPU Satuaration

```shell
sysbench cpu --threads=3 --time=0 run
```

5. Killing Patroni on the primary
6. Manual failover(unplanned event) and switchover(planned activity)
   - What is te difference between two?
8. 

```shell
# node 1
sudo patronictl -c /etc/patroni/config.yml switchover
sudo patronictl -c /etc/patroni/config.yml failover
```
