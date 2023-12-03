Agendas:

1. [Partitioning](#partitioning)
2. [Indexing](#indexing)

![tobias-fischer-PkbZahEG2Ng-unsplash](https://github.com/Mohsem35/DBA/assets/58659448/afb233c6-e653-4398-87db-493cb5e26286)

## Partitioning


#### Benefits of partitioning
1. Partitioning can offer several advantages for your database testing and development, such as **`faster queries`**, **`easier maintenance`**, and **`higher availability`**.
2. **`By dividing a large table into smaller ones`**, you can **`reduce`** the amount of data that needs to be **`scanned`**, **`sorted`**, or **`joined`**. This can improve the response time and throughput of your queries, especially if you use indexes and partitions wisely.
3. Partitioning can also **`simplify the tasks of loading, updating, deleting, or archiving data`**. You can perform these operations on individual partitions instead of the whole table, which can reduce the locking, logging, and resource consumption.
4. You can also use partitioning to **`distribute data`** across different _servers, disks, or locations to improve load balancing and fault tolerance_

#### Drawbacks of partitioning
1. You must choose the right partitioning scheme, strategy, and function based on data characteristics and query patterns.
2. Additionally, you need to monitor and manage the partitions, indexes, and statistics for optimal performance and consistency.
3. Partitioning can also **`restrict`** options for _querying and modifying data, requiring queries or procedures to be rewritten or refactored to use partition elimination or pruning_.
4. Furthermore, it may be **`difficult to use foreign keys, triggers, or constraints`** across partitions or tables.
5. Lastly, partitioning can increase the cost of storage, memory, and licensing of your database system. You may need more disk space and RAM to store and access the partitions and indexes as well as pay extra fees or upgrade your edition for advanced partitioning features
or options.


#### Q: when we should go for table partitioning in a database?


Remember that while partitioning can offer significant benefits, it's **`not a one-size-fits-all solution`**. It's important to carefully consider your specific use case, the nature of your data, and the types of queries you're running before implementing partitioning. Additionally, the effectiveness of partitioning can vary depending on the database management system (DBMS) you're using. Always consult the documentation and consider conducting performance tests before making a final decision.


- Decide when your table is getting "too" big, there are no hard and fast rules as to when a table is getting too big and you should consider partitioning it. Quoting severnines.com again, you should look for pattern when someone says "_I can't do this task because the table is too
big_".
- Table is getting **`bloated with updated and deleted rows`**, running vacuum on large tables will obviously take more time, so if your table is updated frequently, vacuuming will keep taking more and more time if it keeps getting bigger (the table), so decide on this criteria
  as well.


When Should You Consider Partitioning?
As we said above, partitioning can be very powerful, but it is certainly not a good idea for every single use case. Contrary to what people often think, the decision to partition a PostgreSQL table is not strictly based on an absolute table size but rather on various factors 
that interact with the table's size. It's essential to evaluate your database's characteristics and requirements before implementing partitioning.

Generally speaking, you should start thinking about partitioning if you identify with one (or more) of the following:

1. **`You have large tables`**. As we said above, the size of your tables is not the only thing that determines if you may see benefits from partitioning; this said, if you have large tables (_this is from tens of millions of rows to billions of rows_), you would probably
   benefit from partitioning.
2. **`Your ingestion rate is very high`**. Even if the current table size isn't massive, a _high data ingestion rate can indicate that the table will grow significantly in the near future_. To implement a partitioning strategy, it might be beneficial to preemptively manage
  this growth before it starts affecting your performance and maintenance operations.
3. **`You’re beginning to notice query performance degradation`**. Partitioning may also be beneficial if your queries are starting to slow down, especially those that should only touch a subset of your data. This could be true even when your tables are smaller due to the
  complexity of the data and queries. For example, _partitioning can significantly enhance query performance when your daily queries include searches based on a specific range or criteria_. Let's say you're dealing with time-series data: partitioning by date can help you quickly retrieve
  records within a particular time frame without scanning the entire table
4. **`You’re dealing with maintenance overhead`**. As a table grows, maintenance operations like `VACUUM`, `ANALYZE`, and `indexing` can _take longer and might start impacting your operational efficiency_. Partitioning can simplify these operations because you can focus on
  maintaining smaller partitions independently, reducing the impact on your database's overall performance.
5. **`You’re managing data retention policies`**. If your dataset has built-in obsolescence, where _older data is periodically purged_, partitioning can make these operations much more efficient. _Dropping an old partition is much faster and less resource-intensive than deleting rows_.
6. **`You want to use less memory`**. If you want to operate with limited memory, you might benefit from partitioning, as smaller indexes and data chunks fit better in memory and improve cache hit rates. In most cases, this will also improve performance.

## Indexing


#### What is Indexing?

Indexing makes **`columns faster`** to query by **`creating pointers`** to where data is stored within a database.

Imagine you want to find a piece of information that is within a large database. To get this information out of the database the computer will look through every row until it finds it. If the data you are looking for is towards the very end, this query would take a long time to run.


![BasicSearchGif](https://github.com/Mohsem35/DBA/assets/58659448/8c110b0c-d5f8-44cf-86b8-d6b6c1709579)

![BinarySearchGif](https://github.com/Mohsem35/DBA/assets/58659448/8043460f-8ff7-4ada-a9d1-fe673821f5d4)

#### What Exactly is an Index?

An index is a **`structure`** that holds the field the index is sorting and a **`pointer from each record to their corresponding record`** in the original table where the data is actually stored. Indexes are used in things like a contact list where the data may be physically stored in the order you add people’s contact information but it is easier to find people when listed out in alphabetical order.


Let’s look at the index from the previous example and see how it maps back to the original Friends table:

<img width="726" alt="Index_pointsTo_table" src="https://github.com/Mohsem35/DBA/assets/58659448/2925f073-eb22-4feb-b142-b2b613a7a7e7">


Shows how an index is structured relative to the table

We can see here that the table has the data stored ordered by an incrementing id based on the order in which the data was added. And the Index has the names stored in alphabetical order.

#### Types of Indexing

There are two types of databases indexes:

1. Clustered
2. Non-clustered

Both clustered and non-clustered indexes are **`stored and searched as B-trees`**, a _data structure similar to a binary tree_. A B-tree is a “self-balancing tree data structure that maintains sorted data and allows searches, sequential access, insertions, and deletions in logarithmic time.” Basically it **`creates a tree-like structure`** that sorts data for quick searching.

<img width="493" alt="binaryTreeImage" src="https://github.com/Mohsem35/DBA/assets/58659448/9a685af8-7ca5-450a-8e35-63978155b324">


Shows an image of a B-tree's structure

Here is a B-tree of the index we created. Our _smallest entry is the leftmost entry_ and our _largest is the rightmost entry_. All queries would start at the top node and work their way down the tree, if the target entry is **`less`** than the current node the **`left path is followed`**, if **`greater`** the **`right path`** is followed.


**Clustered Indexes**

Clustered indexes are the **`unique index`** per table that **`uses the primary key`** to organize the data that is within the table. The clustered index **`ensures that the primary key is stored in increasing order`**, which is also the order the table holds in memory.

- Clustered indexes **`do not have to be explicitly declared`**.
- Created **`when the table is created`**.
- Use the primary key sorted in **`ascending order`**.

The clustered index will be automatically created when the primary key is defined:

```
CREATE TABLE friends (
    id INT PRIMARY KEY,
    name VARCHAR,
    city VARCHAR
);
```

The created table, “friends”, will have a clustered index automatically created, organized around the Primary Key “id” called **`“friends_pkey”`**

<img width="947" alt="pkeyToTable" src="https://github.com/Mohsem35/DBA/assets/58659448/e99df2d1-be1e-4f3f-95fd-2fd88dc5b343">

However, _in order to search for the “name” or “city” in the table_, we would have to **`look at every entry because these columns do not have an index`**. _This is where non-clustered indexes become very useful_.


**Non-Clustered Indexes**

Non-clustered indexes are **`sorted`** references **`for a specific field`**, from the main table, that **`hold pointers back to the original entries`** of the table. 
They are used to **`increase the speed of queries`** on the table by creating columns that are more easily searchable. Non-clustered indexes can be created by data analysts/ developers after a table has been created and filled.

> _**Note**_ : Non-clustered indexes are not new tables. Non-clustered indexes hold the field that they are responsible for sorting and a pointer from each of those entries back to the full entry in the table.

You can think of these just like **_indexes in a book_**. The _index points to the location in the book where you can find the data you are looking for_.

<img width="467" alt="Index" src="https://github.com/Mohsem35/DBA/assets/58659448/ab7045f8-e8aa-465e-9c31-dc4b850af401">

You can create many non-clustered indexes. As of 2008, you can have up to 999 non-clustered indexes in SQL Server and **`there is no limit in PostgreSQL`**.


**When to use Indexes**

Indexes are meant to speed up the performance of a database, so use indexing whenever it significantly improves the performance of your database. As your database becomes larger and larger, the more likely you are to see benefits from indexing.


**When not to use Indexes**

When data is written to the database, the original table (the clustered index) is updated first and then all of the indexes off of that table are updated. Every time a write is made to the database, the indexes are unusable until they have updated. **`If the database is constantly receiving writes then the indexes will never be usable`**. This is why indexes are typically applied to databases in **`data warehouses`** that get new data updated on a **`scheduled basis`**(off-peak hours) and not production databases which might be receiving new writes all the time.

- **`Columns with Low Cardinality`**: If a column has a low number of distinct values (low cardinality), an index might not be very effective. For example, a column indicating gender in a large dataset might not benefit much from an index because there are only two possible values.
- **`Temporary Data`**: Indexes might not be necessary for tables that hold temporary or staging data that is used briefly and then discarded.

- **`Indexes on Frequently Joined Columns`**: In some cases, creating an index on a column used in frequent JOIN operations can improve performance. However, if the joins are infrequent or if the table is small, the index might not be necessary.
