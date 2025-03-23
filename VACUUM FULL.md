```sql
vacuum full analyze <table_name>;
```



**Transaction ID Wraparound**

Every transaction in PostgreSQL is assigned a **unique identifier** called a **Transaction ID (XID)**, which is similar to numbering each page in a notebook. 

A transaction refers to a set of one or more SQL operations executed as a single unit of work, such as inserting, updating, or deleting data. 

This unique ID helps PostgreSQL keep track of the changes made during the transaction and ensures data consistency. **PostgreSQL uses a 32-bit integer** for these XIDs, meaning that there are **approximately 4 billion possible unique transaction IDs**. 

Max Transaction IDs=2^32 ≈ 4,294,967,296

Once this limit is reached, the system starts reusing IDs, which could cause confusion if not properly managed. This is also another duty for VACUUM to cleanup the older XIDs 

To illustrate the potential problem imagine you’re using a notebook where you number each page to keep track of your notes. If you keep filling up pages and eventually run out of numbers, you might start reusing old page numbers. This would create confusion because you wouldn’t know whether you’re referring to a fresh note or an old one. 

Similarly, **When PostgreSQL reaches this limit, it wraps around and starts reusing IDs**. Without careful management, PostgreSQL could mistake an old transaction for a new one due to ID reuse, leading to data inconsistencies.


**Q: How does PostgreSQL prevent Transaction ID Wraparound?**

By vacuuming. Vacuuming marks old transactions so they don’t interfere with new ones, ensuring PostgreSQL can reuse transaction IDs safely without confusion.

**Q: How to Monitor Transaction ID Wraparound?**

We can run the following query to check the database age and the percentage of progress toward emergency autovacuum (when old tuples are forcefully frozen) and `wraparound_risk_percent` (when PostgreSQL might set the database to read-only to perform vacuum operations).

```sql
SELECT 
    Datname,
    age(datfrozenxid) AS database_age,
    100 * (age(datfrozenxid) /
current_setting('autovacuum_freeze_max_age')::float) AS
Emergency_autovacuum_percent,
    (age(datfrozenxid)::numeric / 2000000000 * 100)::numeric(4,2) AS
Wraparound_risk_percent
FROM 
    pg_database 
ORDER BY 
    age(datfrozenxid) DESC;
```

To check table-wise age run the following query

```sql
SELECT 
    c.oid::regclass AS table_name,
    age(c.relfrozenxid) AS table_age,
    100 * (age(c.relfrozenxid) /
current_setting('autovacuum_freeze_max_age')::float) AS
Emergency_autovacuum_percent,
    pg_size_pretty(pg_total_relation_size(c.oid)) AS table_size
FROM 
    pg_class c
JOIN 
    pg_namespace n ON c.relnamespace = n.oid
WHERE 
    relkind IN ('r', 't', 'm') -- Filter for tables, toast tables, and
materialized views
    AND n.nspname NOT IN ('pg_toast', 'pg_catalog', 'information_schema') -- Exclude system schemas
ORDER BY 
    age(c.relfrozenxid) DESC 
LIMIT 100;
```

**Q: Is Vacuuming Automatic or Manual?**

In PostgreSQL, vacuuming can be both automatic and manual. **Autovacuum is a built-in feature** that runs in the background to clean up dead rows and prevent table bloat.

**Autovacuum reclaims storage and updates statistics without user intervention**, keeping the database performant. However, manual vacuuming is available for when you want more control. You can run the VACUUM command to clean specific tables or manage tables with heavier workloads.
