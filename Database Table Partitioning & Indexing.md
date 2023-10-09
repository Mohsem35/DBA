## Partitioning


#### Benefits of partitioning
1. Partitioning can offer several advantages for your database testing and development, such as **`faster queries`**, **`easier maintenance`**, and **`higher availability`**.
2. _By dividing a large table into smaller ones_, you can **`reduce`** the amount of data that needs to be **`scanned`**, **`sorted`**, or **`joined`**. This can improve the response time and throughput of your queries, especially if you use indexes and partitions wisely.
3. Partitioning can also **`simplify the tasks of loading, updating, deleting, or archiving data`**. You can perform these operations on individual partitions instead of the whole table, which can reduce the locking, logging, and resource consumption.
4. You can also use partitioning to **`distribute data`** across different _servers, disks, or locations to improve load balancing and fault tolerance_

#### Drawbacks of partitioning
1. You must choose the right partitioning scheme, strategy, and function based on data characteristics and query patterns.
2. Additionally, you need to monitor and manage the partitions, indexes, and statistics for optimal performance and consistency.
3. Partitioning can also **`restrict`** options for _querying and modifying data, requiring queries or procedures to be rewritten or refactored to use partition elimination or pruning_.
4. Furthermore, it may be difficult to use _foreign keys, triggers, or constraints_ across partitions or tables.
5. Lastly, partitioning can increase the cost of storage, memory, and licensing of your database system. You may need more disk space and RAM to store and access the partitions and indexes as well as pay extra fees or upgrade your edition for advanced partitioning features
or options.


#### Q: when we should go for table partitioning in a database?


Remember that while partitioning can offer significant benefits, it's not a one-size-fits-all solution. It's important to carefully consider your specific use case, the nature of your data, and the types of queries you're running before implementing partitioning. Additionally, 
the effectiveness of partitioning can vary depending on the database management system (DBMS) you're using. Always consult the documentation and consider conducting performance tests before making a final decision.


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
