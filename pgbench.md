
#### pgbench for PostgreSQL benchmark

pgbench measures database performance by **simulating a workload** of multiple clients executing transactions on a PostgreSQL database. It measures the following metrics:

1. **`Transactions per second (TPS)`**: The number of transactions that can be completed per second. This is a key metric for measuring the overall throughput of the database.
2. **`Latency`**: The time it takes to complete a transaction. This is a measure of the responsiveness of the database.
3. **`Buffer cache hit ratio`**: The percentage of data that is retrieved from the buffer cache instead of from disk. This is a measure of the efficiency of the database's caching system.

pgbench can also measure other metrics, such as the number of bytes read and written, the number of I/O operations, and the CPU usage.

pgbench is a useful tool for benchmarking PostgreSQL databases and comparing their performance under different workloads and configurations.

pgbench - run a benchmark test on PostgreSQL

pgbench is program for **running benchmark tests on PostgreSQL**
It **executes the same sequence of SQL** in multiples of numebrs as specified. It does it in **multiple concurrent database sessions**. and then **calculates** the average transaction rate (**transactions per second**). By default, pgbench tests a scenario that is loosely based on TPC-B involving five `SELECT`, `UPDATE`, and `INSERT` commands per transaction.
You can test other cases by writing your own transaction script files.

```shell
sudo apt-get install postgresql-contrib 
```

After installing the utitity, you can now check all the options in the tool by asking for the help, wherever in the psql terminal or in the sudo terminal by:

```shell
pgbench -? [--help]
```
**Getting Started and work with pgbench**

Before working with pgbench, it recommended to **intialize the test** you will start and the size of this test where pgbench is depend on the TPC-B transaction benechmarking. 

pgbench should be invoked with the **`-i`** (initialize) option to **create and populate these tables**. (When you are testing a custom script, you don't need this step, but will instead need to do whatever setup your test needs.) Initialization looks like:

```shell
pgbench -i [ other-options ] dbname	
```

**`-i`**: This option tells pgbench to initialize the database
**`-s`** - is the database size which will do the test on this database, and database size is reflecting the number of records and tuples in the database tables'.This paramter is called the scaling factor. 
**`dbname`** - is the database name. 
**`-h`** and **`-p`** - the server socket if it is a remote server
**`-U`** - database owner or the role name
**`-c`** number of clients
**`-j`** 2 number of threads
**`-t`** 100 amount of transactions

```
psql -i -h <host_machine_ip> -p 5432 -U postgres -d <database_name> -s 10 -F 20
```
The `-s` option is used to **multiply the number of rows** entered into each table. In the command above, we entered a **“scaling”** option of `50`. This told pgbench to create a database with 50 times the default size.

This means that our `pgbench_accounts` table now has `5,000,000` records. It also means our database size is now `800MB (50 x 16MB)`. If you want to increase the database size more you should use the **`-F`** fillFactor and it should be usually larger than the scalling factor. 


```sql
CREATE OR REPLACE FUNCTION count_rows(schema text, tablename text) RETURNS INTEGER
AS
$body$
DECLARE
  result INTEGER;
  query VARCHAR;
BEGIN
  query := 'SELECT count(1) FROM ' || schema || '.' || tablename;
  EXECUTE query INTO result;
  RETURN result;
END;
$body$
LANGUAGE plpgsql;
```
```sql
SELECT
  table_schema,
  table_name,
  (SELECT count_rows(table_schema, table_name)) AS row_count
FROM
  information_schema.tables
WHERE
  table_schema NOT IN ('pg_catalog', 'information_schema') AND
  table_type = 'BASE TABLE'
ORDER BY
  row_count DESC;
```
```shell
 table_schema |    table_name    | count_rows 
--------------+------------------+------------
 public       | pgbench_accounts |    1500000
 public       | pgbench_history  |      31144
 public       | pgbench_tellers  |        150
 public       | pgbench_branches |         15
(4 rows)

```
```
pgbench -h 10.10.1.200 -p 5432 -c 10 -j 2 -t 10000 <db_name>
```

Typical output from pgbench looks like:

```shell
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 10
number of threads: 1
number of transactions per client: 1000
number of transactions actually processed: 10000/10000 
tps = 85.184871 (including connections establishing) 
tps = 85.296346 (excluding connections establishing)
```


Locating pgbench

```
pgbench -i [other-options] dbname
```

Creating pgbench sample database

For regression testing use a small database (one that fits in `shared_buffers`) that has a scale greater than the number of cores on your system, then run the select-only test on that.

```sql
SET shared_buffers='256MB';
```

Create a database with a scale of 10 (approximately 160MB):

```sql
CREATEDB pgbench
```
```
pgbench -i -s 10 pgbench
```
```shell
export PATH=$PATH:/usr/pgsql-12/bin/
```
```sql
DROPDB pgbench;
CREATEDB pgbench;
```
```
pgbench -i pgbench
```
```
postgres=# \c pgbench
You are now connected to database "pgbench" as user "postgres".
pgbench=# \dt+
```
﻿

#### Creating a baseline
```shell
[postgres@ip-172-31-3-54 bin]$ pgbench -c 50 -j 2 -t 100 pgbench 
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
number of transactions per client: 100
number of transactions actually processed: 5000/5000 latency average 79.025 ms
tps = 632.714730 (including connections establishing) 
tps = 633.083474 (excluding connections establishing)
```

`-c` number of clients
`-j` 2 number of threads
`-t` 100 amount of transactions

These values are 5000 transactions per client. So 50 x 100 = 50,000 transactions

**What is the “Transaction” Actually Performed in pgbench?**

pgbench executes test scripts chosen randomly from a specified list. They include built-in scripts with `-b` and user-provided custom scripts with -f. Each script may be given a relative weight specified after a @ so as to change its drawing probability. The default weight is 1. Scripts with a weight of 0 are ignored.

The default built-in transaction script (also invoked with -b tpcb-like) issues seven commands per transaction over randomly chosen aid, tid, bid and balance. The scenario is inspired by the `TPC-B` benchmark, but is not actually TPC-B, hence the name.


```
1. BEGIN;

2. UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid;

3. SELECT abalance FROM pgbench_accounts WHERE aid = :aid;

4. UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid;

5. UPDATE pgbench_branches SET bbalance = bbalance + :delta WHERE bid = :bid;

6. INSERT INTO pgbench_history (tid, bid, aid, delta, mtime) VALUES (:tid, :bid, :aid, :delta, CURRENT_TIMESTAMP);

7. END;
```



##### script that creates a database cluster with mentioned parameters for Centos 7

```shell
mkdir -p /var/lib/pgsql/13/data1

export PGDATA=/var/lib/pgsql/13/data1
export PGLOG=/var/lib/pgsql/13/data1/log1 
export PATH=$PATH:/usr/pgsql-13/bin/
```

```
initdb
```


```
SHARED_BUFFERS="256MB"
WAL_BUFFERS="16MB"
echo \# Changes for pgbench testing >> $PGDATA/postgresql.conf 
echo shared_buffers=$SHARED_BUFFERS >> $PGDATA/postgresql.conf 
echo wal_buffers=$WAL_BUFFERS >> $PGDATA/postgresql.conf

pg_ctl start -1 $PGLOG
```

[PGBench](https://github.com/AbdallahCoptan/PostGreSQL-Bench/blob/master/Pgbench.md)