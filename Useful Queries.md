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
- kill all connections
```shell
postgres=# SELECT 
    pg_terminate_backend(pid) 
FROM 
    pg_stat_activity 
WHERE 
    -- don't kill my own connection!
    pid <> pg_backend_pid()
    -- don't kill the connections to other databases
    AND datname = 'ibprod'
    ;
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
- Insert exactly same data from existing rows except for 2 colums

```sql
-- add 'BV' to existing ids
-- change profile to 'buet-vetting'
INSERT INTO common.properties
(id, application, profile, "label", "key", "value")
SELECT id || 'BV', application, 'buet-vetting', "label", "key", "value"
FROM common.properties
where profile = 'test';
```

- [Eliminating Duplicate Entries From PostgreSQL](https://medium.com/@nidhig631/delete-duplicate-records-from-the-table-when-all-duplicate-rows-have-the-same-value-32a8973eedd0)

```sql
delete from <table_name> a using <table_name> b where a=b and a.ct<column_name>>b.ct<column_name>;
```

### Errors in MRA

```shell
ubuntu@MRA-project:~$ pg_dump --schema-only -U mra -d mra -n common > mra_schemaonly_common.sql
pg_dump: error: query failed: ERROR:  permission denied for schema common
pg_dump: error: query was: LOCK TABLE common.bank IN ACCESS SHARE MODE
```
solution

```sql
sudo pg_dump --schema-only -U postgres -d mra -Fc -n common > mra_schemaonly_common.sql
```

