pg_repack is a **PostgreSQL extension** which lets you **remove bloat from tables and indexes**, and optionally **restore the physical order of clustered indexes**. Unlike CLUSTER and VACUUM FULL it **works online**, without holding an exclusive lock on the processed tables during processing. pg_repack is efficient to boot, with performance comparable to using CLUSTER directly.


You can choose one of the following methods to reorganize:

1. Online CLUSTER (ordered by cluster index)
2. Ordered by specified columns
3. Online VACUUM FULL (packing rows only)
4. Rebuild or relocate only the indexes of a table

**NOTICE**

- Only superusers can use the utility.
- Target table must have a PRIMARY KEY, or at least a UNIQUE total index on a NOT NULL column.


**Advantages of using Pg_repack**

Release Storage from a table to disk/FS.
Rebuild a table. Reduces I/O.
Release storage piled due Dead tuples as Bloat and not cleared by AV.

#### How to Install pg_repack extension

Step1: Getting stated with installation

```shell
# sudo apt install postgresql-12-repack     ubuntu
yum install pg_repack12
sudo su postgres
psql
```
```sql
SET  shared_preload_libraries = 'pg_repack';
```
Restart the databse cluster
```shell
# sudo systemctl restart postgresql         ubuntu
/usr/pgsql-12/bin/pg_ctl -D $PGDATA restart -mf
```

Step2: Create this extension in database/s where you are planning to execute it.

```shell
sudo su postgres
psql
```
```sql
# \c <database_name>
c dvdrental;
CREATE EXTENSION pg_repack;
\q
```
> Pg_repack works with individual database. spo you have to declare in the database 

Step3: pg_repack utility to Rebuild Tables Online


```shell
echo "export PATH=$PATH:/usr/pgsql-12/bin" >> ~/.bashrc
source ~/.bashrc
```


```shell
# /usr/pgsql-12/bin/pg_repack-dry-run -d <database_name> --table <table_name>
/usr/pgsql-12/bin/pg_repack-dry-run -d dvdrental --table actor

# pg_repack -d <database_name> table <table_name>
/usr/pgsql-12/bin/pg_repack -d dvdrental table actor
```


Step4. How to Rebuild an entire Database using pg_repack

```shell
pg_repack --dry-run -d dvdrental        # (dry run )
Pg_repack -d dvdrental -j 2             # (actual run)
```

Note: This can be run remotely also, only condition is version of pg_repack must be same.

**Appendix**

```shell
DROP EXTENSION pg_repack;
yum remove pg_repack12
```
