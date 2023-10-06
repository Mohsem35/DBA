## How do you fix slow databases in production


### Identify the root cause 

In order to fix a slow database, the first step is to identify the root cause of the issue. To do this, **`monitor and measure the performance of your database`**, and look for any signs of bottlenecks, 
errors, or anomalies. You can utilize various tools and methods to collect and analyze data, such as `database logs`, `metrics`, `queries`, and `indexes`. Database logs record transactions, queries, and 
events in your database. Database metrics show the status, health, and activity of your database. Database queries are commands sent to your database to manipulate and retrieve data. And database indexes 
are data structures that help your database find and access data faster. By recognizing the root cause of a slow database, you can focus your troubleshooting and optimization efforts.


**_Michael C.(Database Engineer at GETCHOICE!_)**

Utilizing the **`Query Store`** is the best way to _identify your slowest-performing queries_. 
To enable the query store utilize this code: 

```sql
ALTER DATABASE <database_name>
SET QUERY_STORE = ON;
```
The following query _returns information about queries and plans in the Query Store_

```sql
SELECT Txt.query_text_id, Txt.query_sql_text, Pl.plan_id, Qry.*
FROM sys.query_store_plan AS Pl
INNER JOIN sys.query_store_query AS Qry
    ON Pl.query_id = Qry.query_id
INNER JOIN sys.query_store_query_text AS Txt
    ON Qry.query_text_id = Txt.query_text_id;
```
This query provides a huge rabbit hole of information that can't be described in 750 characters or less. There are literal books written on this subject.



_Adding more memory to your system will allow you to convert your most utilized tables_ to **`Memory Optimized Tables (MOTs)`**. MOTs perform on average **`9x faster`** than traditional tables in most instances. 
There is a bit of maintenance associated with MOTs, such as `BUCKET_COUNT` increases, but this can be automated with some fairly simple Server Jobs in MSSQL (or Logic Apps in Azure SQL). There is also some special parameters you must use when joining a MOT to a traditional table if you have snapshot isolation enabled. 

Example:

```sql
select * from MOT_Table WITH (SNAPSHOT)
join Traditional_table on Traditional_table.mot_id = MOT_Table.id
```


**_Arockia Nirmal Amala Doss(Data Engineer AWS | SQL | PYTHON | DATA WAREHOUSING | ETL | CI/CD | IaC | AWS Community Builder)_**

- Utilize Automated Performance Monitoring tools to detect issues early.
- Employ Database Tuning Advisors for query optimization suggestions.
- Implement real-time dashboards to monitor key database metrics.
- Analyze database logs, metrics, queries, and indexes meticulously to pinpoint anomalies.
- Conduct load testing to identify performance bottlenecks under different traffic conditions.
- Engage in thorough code reviews to uncover inefficient queries.
- Seek external expertise or consult database vendors for specialized diagnostic tools or advice

### Optimize your queries

**`Poorly written or inefficient queries`** are a _common cause of slow databases_. Queries that are too `complex`, `frequent`, or `large` can consume significant resources and time, thus affecting the 
performance of your database and applications. To ensure optimal performance, you should use **`appropriate data types and formats`** for your columns and variables, while **`avoiding implicit conversions 
or casts`**. Additionally, use filters, joins, and subqueries wisely to reduce complexity and cost. Indexes can be used to speed up search and retrieval of data; however, _be aware of over-indexing or 
under-indexing your table_. Pagination, batching, or caching techniques can also be employed to limit the amount of data returned or processed by queries, thus reducing network traffic and memory usage. 
By optimizing your queries, you can improve the performance and scalability of your database and applications while reducing errors or timeouts.


**_Iftikhar Shoukat(Database Administrator at United Bank Limited)_**

Here are below few things to get started,

1. Check longops
2. Identify the queries which are causing full table scan
3. Rebuild the index if they are invalid
4. Highlight the queries to developer and make necessary index wherever required 
5. Update the latest stats of table and index


**_Vinícius Bazana Schmidt(Database and Software Engineer @ Database Performance and PostgreSQL Specialist)_**

On databases such as PostgreSQL, you can leverage extensions such as **`pg_stat_statements`** which will track all queries and store procedures saving how much time it takes to execute and the average time. 

With that information, you can focus on the right queries that you want to tune - making use of **`EXPLAIN ANALYZE`** to understand the **`execution plan of that query`**. Reading the plan can be tricky 
depending on the query size - but it's the best way to understand **`how the query is operating behind the scenes`** (_which indexes are being used, if there are full table scans, etc_). 
Those details will give you the right bootstrap to tune your queries and create the right indexes. Another pro-tip is to use partial indexes (with WHERE clause).

### Tune your parameters

Slow databases are often caused by suboptimal or outdated configuration parameters. Parameters are settings that control the behavior and performance of your database, such as **`memory allocation`**, 
**`concurrency control`**, **`connection pooling`**, **`logging level`**, etc. To tune your parameters, you can review your current values and compare them to the recommended or default values for your database version 
and platform. Additionally, identify any parameters that are critical or relevant for your database performance and prioritize them based on their impact and feasibility. Afterward, test and measure the 
effects of changing your parameter values on your database performance with tools such as _benchmarks, baselines, or simulations_. 


**_Asif Butt(IT Manager at Pakistan Kidney & Liver Institute And Research Center)_**

- Performance Monitoring: Use tools to monitor and identify bottlenecks and resource usage.
- Database Indexing: Ensure proper indexing for tables and optimize them based on query patterns.
- Query Optimization: Review and optimize SQL queries, avoiding SELECT * and retrieving only necessary data.
- Connection Pooling: Implement connection pooling to reduce connection overhead.
- Caching: Use caching to store frequently accessed data in memory for read-heavy workloads.
- Schema Optimization: Review and optimize the database schema.
- Load Testing: Test your database under stress to proactively identify issues.Query Caching: Implement query caching in your application.


**_Mohsen Bahadori(Senior Oracle Database and Linux Administrator)_**



First, the level of the problem must be determined. Is the problem `database-wide` or a `subset of the system` or `just one query`? This helps us **`categorize`** the level of the problem and also find the 
right person from the application or business to contact. Now you can look for the root cause in the right place. The scope of the cases to be investigated is more specific. For example, to check a slow 
query, we can check the statistics related to the query objects. For slowness of a subset of queries, we can check some parameters from the App team and find the common places of all of them and in case of 
slowness in the whole system, we have to check common parameters in OS, App, and DB.
