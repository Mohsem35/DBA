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



**`Normalization:`** is a technique that **`divides tables into smaller`** tables and connects them via relationship, Goal is to **`reduce redundancy`** and **`data integrity`**. The basic idea is to 
identify repeating groups of data and creating new tables for them

- _Star_ schema: Denormalized
- _Snowflake_ schema: **`Normalized`**. extension of star schema, here the dimension table is more normalized)

**`1NF`**: Each record must be **`unique`**, with no duplicate rows, and each cell must hold one value
**`2NF`** - must satisfy 1NF, if the **`primary key is one column then auto satisfies 2NF`**, if composite primary key then non-key column must be dependent on all the keys. 
All non-key columns are dependent on the table's PRIMARY KEY
