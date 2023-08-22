## Recursive Common Table Expressions
When you **`reference`** a Common Table Expression **`within itself`**, it becomes a recursive Common Table Expression.

A Recursive Common Table Expression, as the name implies, is a common table expression that can **`run a subquery multiple times`**, as long as a condition is met. 
It iterates continuously **`until it reaches a breakpoint`** when the condition stops being true.

- To define a recursive CTE, the **`RECURSIVE keyword`** must be in its name. Without this keyword, MySQL throws an error.


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

The subquery is in two parts, joined by the UNION ALL keyword to form one. You can also join these subqueries by the **`UNION`** keyword if you don't need **`duplicate 
records`**.

So at the start of the iteration, `n` is 1 and `square` is also 1. That means, `n + 1` is 2, and `(n + 1) * (n + 1)` is 2 *2 which is 4. 2 and 4 get added to the result 
table and then become the most recent values in the table. `n` becomes 2, and `square` becomes 4.

This continues until the condition in the WHERE keyword is stops being true.

![Screenshot-2023-02-17-at-23 55 27](https://github.com/Mohsem35/DBA/assets/58659448/cdd820fa-fb4f-4b69-8d38-5104e5bac3c5)

The recursion continues until the value of 'n' reaches 10.




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
