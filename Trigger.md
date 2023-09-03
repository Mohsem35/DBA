A trigger is a **`set of SQL statements`** that **`reside in system memory`** with unique names. 
- It is a specialized category of stored procedure that is **`called automatically`** when a database server **`event occurs`**. 
- Each trigger is always **`associated with a table`**.
- Triggers have **`no`** chance of **`receiving parameters`**.
- A transaction cannot be committed or rolled back inside a trigger.

A trigger is called a special procedure because it **`cannot be called directly`** like a stored procedure. The key distinction between the trigger and procedure is that 
a trigger is called automatically when a data modification event occurs against a table. A stored procedure, on the other hand, must be invoked directly.

## Syntax of Trigger

```
CREATE TRIGGER schema.trigger_name
BEFORE/AFTER
{INSERT, UPDATE, DELETE}
ON table_name
AS
{SQL_Statements}
```
