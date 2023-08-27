- _Column_ = Field, _Row_ = Record
- **`SQL vs MySQL`**: SQL হল language for **`Data manipulate`** এবং update in a Database. অন্যদিকে MySQL হলো **`database management system`** যেখানে data organized way তে থাকে in a database.
- A database role is an entity that contains information that defines the role's privileges and interacts with the client authentication system.
- `information_schema` is a **`meta-database`** that holds information about your current database


```sql
# will return all of the tables in the database
SELECT
    table_name
FROM
    information_schema.tables
```

```sql
# will return a list of all the columns and tables in the database
SELECT
    table_name, column_name
FROM
    information_schema.columns
```

```sql
# COLUMNS from a specific table and return the column names from the specific table in database.
SELECT
    column_name
FROM
    information_schema.columns
WHERE
    table_name = 'Album'
```
```sql
# Query the right table information_schema to get columns by table_schema
SELECT 
    column_name, data_type 
FROM 
    information_schema.columns 
WHERE 
    table_name = 'university_professors' AND table_schema = 'public';
```

```sql
# TABLE CONSTRAINTS from a specific table and return the column names from the specific table
SELECT
    table_name, constraint, constraint_name, constraint_type
FROM
    information_schema.table_constraints
WHERE
    constraint_type = 'Album'
```
```sql
# view views in the database you are querying.
SELECT * FROM information_schema. Views;
```

- Copy data from an existing table to a new table following `INSERT INTO SELECT DISTINCT` Pattern

```sql
INSERT INTO new_table
SELECT * FROM old_table
```
**`Integrity Constraints()`**: the protocols that a _table's data columns must follow_ (attribute constraints[data types], key[candidate, primary], referential integrity)

- The combination of all attributes is **`super key`**
1. Super key
2. Candidate key
3. Surrogate key

**`CASCADE`**: It is used in conjunction with `ON DELETE` or `ON UPDATE`. It means that the _child data is either deleted or updated when the parent data is deleted or updated_

- OLTP & OLAP (approaches to processing data)

**`OLTP`**: _Online Transaction Processing_. **`good for day-to-day operation`**, application-oriented, snapshot(GB), **`updates frequently`**, high normalized

**`OLAP`**: _Online Analytical Processing_, **`good for business decision making`**, subject-oriented, archive(TB), **`updates limited`**, less normalized

- **`Structure data`**(follows schema, data types, _RDBMS_), **`Non Structure data`**(Schema less, _photos/mp3_), **`Semi-structure data`**(self describing data, _NoSQL/JSON/XML_)
- Data marts - subset of Data warehouse
- Data lakes - stores all types of data at a lower cost like raw, operational,  IoT devices logs, relational/non-relational
- ETL - Extract, Transform, Load   [Writing data into a warehouse is the "Loading" part of ETL]  This python script is the "Transforming" part of ETL, where we clean data to fit a schema.
- ELT - Extract,  Load, Transform

**`Database design`** consists of 2 parts.
  1. Database Models &
  2. Schemas

**`Database models`**: 
  1. _Relational_ model
  2. _Network_ model,
  3. _NoSQL_ model,
  4. _Object-oriented_ model

**`Schemas`**: the **`blueprint`** of the database, defines _tables_, _views_, _fields_, _relationships_

**`Data Modeling`**: 
  1. **`Conceptual`**(describes **`entities, relationships, and attributes`** e.g. ER diagram),
  2. **`Logical`**(describes **`tables, columns, relationships`** e.g. relational model and star schema),
  3. **`Physical`**(describes **`physical storage`**)



**`Normalization:`** is a technique that **`divides tables into smaller`** tables and connects them via relationship, Goal is to **`reduce redundancy`** and **`data integrity`**. The basic idea is to identify repeating groups of data and create new tables for them. Reduce data duplication, increase data consistency and data organization

- _Star_ schema: Denormalized
- _Snowflake_ schema: **`Normalized`**. extension of star schema, here the dimension table is more normalized)

**`1NF`**: Each record must be **`unique`**, with no duplicate rows, and each cell must hold one value

**`2NF`**: must satisfy 1NF, if the **`primary key is one column then auto satisfies 2NF`**, if composite primary key then non-key column must be dependent on all the keys. 
All non-key columns are dependent on the table's PRIMARY KEY

**`3NF`**: must satisfy 2NF, **`no transitive dependencies`**(মানে এইখানে ৩ টা কলাম direct involved হইব। For example `course name > teacher > room number`। মানে _আমি course name জানতে পারলেই teacher এর room number ও জানতে পারব_, ৩ টা কলাম involved করতে পারলাম,  এইটাকেই বলে transitive dependencies.

**`Database views`**: **`virtual table`** that is not part of the physical schema, also known as **`non-materialized views`**. View(virtual table) টেবিলে insert, update সব কিছু করা যাইব

- Aggregation Functions: `SUM()`, `AVG()`, `COUNT()`, `MIN()`, `MAX()`, `GROUP BY`

![rsz_1263541095-711d2b2a-7f5e-42a1-8461-aae7ea29d729](https://github.com/Mohsem35/DBA/assets/58659448/328154bc-d396-4e98-a2e3-bdbcb2e9a90c)

- Joins: `INNER`, `LEFT`, `RIGHT`, `FULL`। Why use joins - Lookup tables, Combine data.

**`Subquery`**:  Join Alternative, returns only a single result instead of a join's many rows. Subquery আমরা করে পারি SELECT, WHERE, FROM দিয়া

**`Common Table Expressions(CTE)`**: Join Alternative, Standalone query with temporary results set, With Statement লাগে, CTE করতে পারি WITH দিয়ে

- Conditionals: `WHERE HAVING`, `UNIQUE`, `NOT NULL`, `AND`, `OR`

**`Materialize views`**: 

**`stores query results`**, not query. **`Faster`**, stores update results after successful execution. Refresh data, `good for data warehouse(OLAP)`. Ex - `Apache airflow`, LUIGI

- Table Partitioning(Physical data layer) - Vertical(Normal Forms, Column ধইরা কাজ করে), Horizontal(২০১৮ সালের ডাটাবেস, ২০১৯ সালের ডাটাবেস, Row ধইরা কাজ করে)
- যখন **`Horizontal table partition`** করতে several machine ব্যবহার করা হয় তখন তাকে বলে **`sharding`**


![rsz_2capture](https://github.com/Mohsem35/DBA/assets/58659448/abe77318-bce1-45fa-a502-4a3350f92bd4)
![rsz_1capture1](https://github.com/Mohsem35/DBA/assets/58659448/f62a55e2-8563-492c-a750-c84fd4436520)


**`DBMS Types`**: - 
1. SQL types,
2. NoSQL types

SQL DBMS: Microsoft SQL Server, PostgreSQL, Oracle

NoSQL DBMS
- _Redis_ (**`key, value`**)
- _MongoDB_ (**`document store`**): good for content management(documents, files, songs, videos)
- _Cassandra_ (**`columnar`**): good for big data analysis where speed is important 
- _neo4J_ (**`graph database`**): good for social media data neo4J

- User Level Schemas = PostgreSQL schemas is the ability to provide database users with their own group of tables that are only accessible to each individual user. মানে main database টা ডিরেক্ট ব্যবহার না করে main database এর মত একটা কপি ডাটাবেস developer দের জন্য, আবার আরেকটা Admin এর জন্য

**`Temporal data type`**: Date, Time, **`Timestamp`**(mostly used), Intervals(hour, minutes) এইসব ডাটা টাইপ রে বুঝায়

**`Boolean data type`**: true, false, null

**`Q: What difference between CHAR, VARCHAR, and TEST data type?`**

Answer: `CHAR` is **`fixed length`** and `VARCHAR` is variable length where `TEXT` is the **`string of variable length`**

- VARCHAR **`without N`** specified equivalent to **`TEXT`**
  
**`Index of SQL`**: Indexes are Special lookup tables. এরা হল **`pointer of data in a table`**. The users cannot see the indexes, they are just used to **`speed up searches/queries`**. 

- Sorted column keys ব্যবহার করে for better search.
- Database Administrator সাধারন্ত indexes ই ব্যবহার করে থাকে
- Indexing ভাল large টেবিলের জন্য
- Column with many nulls is not good for indexing.

- Clustered and Non-clustered indexes are applied to table columns in a database. 
- **`Updating a table with indexes takes more time than updating a table without`** (because the indexes also need an update). সুতরাং শুধুমাত্র frequently search অথবা query করার কাজে ব্যবহার করার জন্যই index use করব আমরা । Ex - `বইয়ের সূচীপত্র -> Various topics with index numbers(লিস্টের সাথে পরিচয়) - Reference of pages`. 

**`Trigger`**: A trigger is a **`special stored procedure`** in a database that **`automatically invokes`** whenever a special event in the database occurs. For example, a trigger can be invoked when a row is inserted into a specified table/when certain table columns are being updated
- `AFTER` trigger mostly ব্যবহার হয় for `auditing and logs`
- Trigger এর alternative হলো  SP, Computed Columns. Triggers are good for - **`auditing`**, integrity enforcement যেখানে SP for general tasks, **`user-specific needs`**

Usage of Triggers: 

1. Enforce business rules by implementing business logic itself
2. **`Validate data`** even before they are inserted or updated
3. Help us to keep a log of records like maintaining audit trails in tables
4. Provide an alternative way to **`run the scheduled task`**
5. Preventing data manipulation
6. Tracking data, or database object changes
7. **`Audit database`**

3 types of Triggers according to T-SQL
- _DML_ triggers(`INSERT`, `UPDATE`, `DELETE`)
- _DDL_ triggers (`CREATE`, `ALTER`, `DROP`)
- _Logon_ triggers (LOGON events)

**`Events`**: MySQL Events are named **`database object`** which contains one or more SQL statement. They are stored in the database and **`executed at one or more intervals`**. Ex - Daily রাতের ১২ টায় database update হবে
- MySQL Event Thread টা আগে Enable করা লাগে, then আমরা events গুলো দেখতে পাই। 
```
SET GLOBAL event_scheduler = ON;
SHOW PROCESSLIST;
```

**`SQL Constraints`**: `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT`, `CREATE INDEX`

**`TRY CATCH`**: TRY block এ কোন error থাকলে CATCH block invokes হবে আর TRY to block এ কোন error না থাকলে CATCH block skipped হবে. Place the error handling block in the Catch block

**`Temp tables`**: short-lived tables. **`যতক্ষন session থাকবে ততক্ষন এই টেবিল টা থাকব`**, copy of a real table whereas virtual table is an object of SQL table

```sql
CREATE TEMP TABLE name AS
```
- টেবিল আসলে slow হয় view logic এর কারনে, কারন table contains data and **`views contain the direction of data`**

**`SQL order of operations`**: 


![Capture (2)](https://github.com/Mohsem35/DBA/assets/58659448/c790a5d9-7829-405e-9c31-8124e63b8e6b)

**`EXPLAIN`** আমরা যে কোন query এর আগে বসাতে পারি to see the **`ordered steps of the execution plan`**. Query Planner is her name. EXPLAIN ২ টা প্যারামিটার। **`EXPLAIN VERBOSE`**(see the available columns at each step of the plan.), **`EXPALIN ANALYZE`**(_computes actual run times in milliseconds_)
```
LIKE ANY translates OR
LIKE ALL translates AND
```
- OR এর জায়গায় IN/ARRAY ব্যবহার করব, আর TEXT use না করে Numeric  use করব, এতে character use হবে কম, execution হবে fast
- Data granularity -  Data's level of detail. The more granular the data, the more detailed it is and the more precise analysis can be. Granularity আমরা তখন করব, যখন data duplicate অথবা data double counting এর সুযোগ না থাকে,  তাহলে minimum results পাওয়ার সম্ভাবনা থাকবে
- The query can reference data FROM Table(base table, temporary table), View(View, Materialized View)
**`Data Store`**:

Database তে ২ ভাবে data store করে রাখে।
1. **`Row oriented`** storage(relation between `columns`)
2. **`Column-oriented`** storage(relation between `rows`)

Row(Horizontal) oriented database `optimization` করা যেতে পারে 2 টা উপায়ে। 
1. by **`Partitions`**
2. by **`Indexes`**

**`Column Oriented`**: good for analytics, transactional database এর জন্য ভালো না


- কিভাবে row number কমায়া আনা যেতে পারে SQL তে?  by using `WHERE`, `LIMIT`, `DISTINCT`, `INNER JOIN`
- Partitions - Partition একটা টেবিলকে অনেকগুলো টেবিলে partition করে। Parent table, Child table
```sql
SELECT * FROM pg_indexes;
```
- Cost estimates  =  time in SQL


**`Concurrency phenomena`**: Dirty reads, Non-repeatable reads, Phantom reads(এই ৩ টা হলো READ UNCOMMITTED এর cons)
- যদি uncommitted data দেখতে চাই তাহলে আমরা READ UNCOMMITTED ব্যবহার করব। এখানে dirty reads = yes, non-repeatable reads = yes, phantom reads = yes
- যদি committed data দেখতে চাই তাহলে আমরা READ COMMITTED ব্যবহার করব। এখানে dirty reads= no, non-repeatable reads = yes, phantom reads = yes
- যদি committed data দেখতে চাই এবং data reading করার সময়ে অন্য transaction যাতে আমার reading data change করতে না চাই তাহলে REPEATABLE READS তে । dirty reads = no, non-repeatable reads = no, phantom reads = yes
- যদি data consistency must চাই তাহলে  SERIALIZABLE। SERIALIZABLE তে আমরা কুয়েরি ২ ভাবে করতে পারি, with Index and without index, এতে করে টেবিল locked হয়ে যায়।  dirty reads = no, non-repeatable reads = no, phantom reads = no
- শুধুমাত্র committed changes গুলোই আমরা SNAPSHOT তে দেখতে পাব, অন্য কোথাও Transaction change হইলে ও একই সময়ে SNAPSHOT update হবে না। SERIALIZABLE এর সাথে SNAPSHOT এর পার্থক্য হলো এইটা টেবিল Locked করে না. dirty reads = no, non-repeatable reads = no, phantom reads = no
- আচ্ছা এখানে Transactions গুলো depends করতেসে BEGIN ->COMMIT/ROLLBACK Transactions এর circle এর উপ্রে 
- On-Premises Database = আমরা পিসি তে hard disk তে যেই data রাখি locally, সেইটাই  on-premises database

**`ODBC`**: Open Database Connectivity একটা **`API`** developed by Microsoft that **`allows applications to access`** data in database management systems **`(DBMS)`** using SQL. ODBC permits maximum interoperability, which means a single application can access different DBMS. The functionality of ODBC is supported by its four components: an application, driver manager, driver, and data source.

- How would you handle data loss during a database migration? - আগে Find out করব data loss টা হইতাসে কোন জায়গা থেইকা, then ওইটা repair করে database backup নিব

**`Hadoop`**: Open source **`framework for big data`**. Financial services companies use analytics to assess risk, build investment models, and create trading algorithms. Hadoop-powered analytics to execute predictive maintenance on their infrastructure

**`Backup`**

Backup type 3
1. Full
2. Differential(যদি আমি আগে full backup নিয়ে রাখি then শুধুমাত্র যেই objects গুলো change হইসে সেইগুলির backup নিতে চাই তাহলে differential)
3. Transaction Log(যদি আগে backup থাকে then changes transactions গুলোর backup নিবে)

**`EDA`**: Exploratory Data Analysis. Iterative process ব্যবহার করব CONVERT এবং DATEPART syntax


- UDF - User-defined function. 1. parameter accept করে 2. perform an action 3. Return result  করে হতে পারে সেইটা table. UDFs are also Modular Programming Tool

What is Modular Programming ? - 1. Software design technique 2.আলাদা আলাদা functionality গুলো independently separate করে ফেলে


A navigational database is structured like a tree. A value has a parent and a child, and that is how they link to each other. If you want to access data, you have to follow a particular route as you move from parent to child. On the other hand, a relational database is more flexible and uses a primary key, which is a unique identifier to access data.

**`ACID`**

Database Transaction must be 
- **`A(atomic)`**: either completed or none of them completed 
- **`C(consistent)`**: all data must be left in a consistent state
- **`I(isolated)`**: transaction must be independent of another transaction
- **`D(durable)`**

- Log shipping is the same as a backup in Microsoft SQL. It uses rapid failover in instances when the main server is not working. Therefore, Log shipping is done manually.
- SQL Server replication is a technology for copying and distributing data and database objects from one database to another and then synchronizing between databases to maintain consistency and integrity of the data. In most cases, replication is a process of reproducing the data at the desired targets


**`CAP theorem`**: 
CAP theorem or Eric Brewers theorem states that we can only achieve at **`most two out of three guarantees for a database`**: Consistency, Availability, and Partition Tolerance.
- **`Consistency`**: means that all nodes in the network _see the same data at the same time_
- **`Availability`**: is a guarantee that _every request receives a response_ about whether it was successful or failed.
- **`Partition Tolerance`**: even if there is a network _outage in the data center and some of the computers are unreachable_, still the _system continues to perform_

Distributed Transaction - ইহা অন্যান্য SQL Transaction এর মত, with a single key difference. The difference is this type of transaction can exist across multiple servers
**`UNION vs UNION ALL`**: Union All will not remove duplicate rows or records


- 2 types of Triggers based on behavior. 1. AFTER trigger 2. INSTEAD OF trigger(prevents execution)
