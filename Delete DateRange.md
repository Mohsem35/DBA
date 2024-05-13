To evacuate the 192.168.190.233 database(cbslog), we need the run this query

> Note: make sure we have a proper backup file


```sql
DELETE FROM public.logintrail 
WHERE signintime >= '2024-03-25'::date AND signintime <= '2024-04-30'::date;
```
