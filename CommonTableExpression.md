
**`A common table expression (CTE)`**:  is a named temporary result set that exists within the scope of a single statement and that can be referred to later within that statement, possibly multiple times.


You'll primarily use a common table expression for two reasons:

- To write queries **`without using subqueries`** (or using fewer subqueries)
- To write **`recursive functions`**


## How to Create a Common Table Expression
You can create a Common Table Expression (CTE) using the **`WITH keyword`**. You can specify multiple common table expressions at the same time by `comma-separating` the queries making up each common table expression.

The general shape of a Common Table Expression is like so:

```
WITH cte_name AS (query)

-- Multiple CTEs
WITH
    cte_name1 AS (
        -- Query here
    ),
    cte_name2 AS (
        -- Query here
    )
```

## CTE Example

```
WITH
    barca_players AS (
        SELECT
            id,
            player_name,
            nationality,
            position,
            -- TIMESTAMPDIFF is a function for calculating players age
            TIMESTAMPDIFF (YEAR, player_dob, CURRENT_DATE) age
        FROM
            wc_players
        WHERE
            club = 'Barcelona'
    )
SELECT
    *
FROM
    barca_players;
```


![Screenshot-2023-02-17-at-22 59 25](https://github.com/Mohsem35/DBA/assets/58659448/f601bddb-7412-4f18-a247-aac72bb2261a)


## How to Use Common Table Expressions With Parameters

You can also pass arguments to the CTE. These are aliases you can use for referencing columns of the query results. The number of parameters passed into the CTE 
must be the same as the number of columns being selected in its subquery. This is because the columns get matched to the aliases one by one, one after the other.

For example, in the barca_players CTE created above, you can decide to refer to the nationality column as country, and position as role:

```
WITH employees_in_california AS (  
    SELECT * FROM employees WHERE city = 'California'   
    )   
    SELECT emp_name, emp_age, city FROM employees_in_california  
    WHERE emp_age >= 32 ORDER BY emp_name;  
```



```
WITH
    barca_players (id, player_name, country, role, age) AS (
        SELECT
            id,
            player_name,
            nationality,
            position,
            TIMESTAMPDIFF (YEAR, player_dob, CURRENT_DATE) age
        FROM
            wc_players
        WHERE
            club = 'Barcelona'
    )
SELECT
    player_name,
    role
FROM
    barca_players;
```

## Recursive Common Table Expressions
When you **`reference`** a Common Table Expression **`within itself`**, it becomes a recursive Common Table Expression.

A Recursive Common Table Expression, as the name implies, is a common table expression that can **`run a subquery multiple times`**, as long as a condition is met. 
It iterates continuously **`until it reaches a breakpoint`** when the condition stops being true.

- To define a recursive CTE, the **`RECURSIVE keyword`** must be in its name. Without this keyword, MySQL throws an error.

#### Example 1
```
# Write a common table expression that prints numbers 1 to 10 and their squares
WITH RECURSIVE
    numbers_list (n, square) AS (
        SELECT
            1,
            1
        UNION ALL
        SELECT
            n + 1,
            (n + 1) * (n + 1)
        FROM
            numbers_list
        WHERE
            n < 10
    )
SELECT
    *
FROM
    numbers_list;
```

The query starts with an initial row (1, 1) and then recursively generates subsequent rows by incrementing the 'n' value and calculating the square of the 
incremented value. 


```
SELECT
    1,
    1
```
After this first query, the result table has one row, and looks like this

![Screenshot-2023-02-17-at-23 55 27](https://github.com/Mohsem35/DBA/assets/58659448/cdd820fa-fb4f-4b69-8d38-5104e5bac3c5)

The subquery is in two parts, joined by the UNION ALL keyword to form one. You can also join these subqueries by the **`UNION`** keyword if you **`don't`** need **`duplicate records`**.

```
SELECT
    n + 1,
    (n + 1) * (n + 1)
FROM
    numbers_list
WHERE
    n < 10
```
So at the start of the iteration, `n` is 1 and `square` is also 1. That means, `n + 1` is 2, and `(n + 1) * (n + 1)` is 2 *2 which is 4. 2 and 4 get added to the result table and then become the most recent values in the table. `n` becomes 2, and `square` becomes 4. This continues until the condition in the WHERE keyword stops being true. The recursion continues until the value of 'n' reaches 10.

- You **`can't use`** the **`GROUP BY`** keyword. This is because you can only group a collection, but in a recursive common table expression, records are handled and evaluated individually. Other keywords like **`ORDER BY`**, **`DISTINCT`**, and aggregate functions like **`SUM`** cannot be used either.


#### Example 2
```
-- generates a series of the first five odd numbers
WITH RECURSIVE odd_num_cte (id, n) AS (
    SELECT 1, 1
    UNION ALL
    SELECT id + 1, n + 2 FROM odd_num_cte WHERE id < 5
)
SELECT * FROM odd_num_cte;
```

#### Example 3
```
WITH RECURSIVE
    fibonacci (n, fib_n, next_fib_n) AS (
        /*
        * n - Number of iterations
        * fib_n - Currennt Fibonnaci number. Starts at 0
        * next_fib_n - Next Fibonnaci number. Starts at 1
        */
        SELECT
            1,
            0,
            1
        UNION ALL
        SELECT
            n + 1,
            next_fib_n,
            fib_n + next_fib_n
        FROM
            fibonacci
        WHERE
            n < 20
    )
SELECT
    *
FROM
    fibonacci;
```
