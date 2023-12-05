```sql
# from psql
VACUUM (VERBOSE, ANALYZE);
```


```sql
# from ABS team
vacuum full analyze public.account;

set client_min_messages = 'warning';
vacuum full;
```

- How many dead tuples are there in a database

```sql
SELECT relname, n_dead_tup FROM pg_stat_user_tables;
```
- Space used by each table

```sql
SELECT relname AS "table_name", pg_size_pretty(pg_table_size (pgc.oid)) AS "space_used"
FROM pg_class AS pgc 
LEFT JOIN pg_namespace AS pgns 
ON (pgns.oid = pgc.relnamespace) 
WHERE nspname NOT IN ('pg_catalog', 'information_schema') AND nspname !~ '^pg_toast' AND relkind IN ('r') 
ORDER BY pg_table_size (pgc.oid) DESC;
```

- Autovacuum executed last time

```sql
SELECT relname, last_vacuum, last_autovacuum FROM pg_stat_user_tables;
```
