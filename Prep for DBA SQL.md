
### Edureka SQL interview questions


_EmployeeInfo Table:_



| EmpID | EmpFname | EmpLname | Department | Project | Address       | DOB        | Gender |
|-------|----------|----------|------------|---------|---------------|------------|--------|
| 1     | Sanjay   | Mehra    | HR         | P1      | Hyderabad(HYD)| 01/12/1976 | M      |
| 2     | Ananya   | Mishra   | Admin      | P2      | Delhi(DEL)    | 02/05/1968 | F      |
| 3     | Rohan    | Diwan    | Account    | P3      | Mumbai(BOM)   | 01/01/1980 | M      |
| 4     | Sonia    | Kulkarni | HR         | P1      | Hyderabad(HYD)| 02/05/1992 | F      |
| 5     | Ankit    | Kapoor   | Admin      | P2      | Delhi(DEL)    | 03/07/1994 | M      |


_EmployeePosition Table:_



| EmpID | EmpPosition | DateOfJoining | Salary |
|-------|-------------|---------------|--------|
| 1     | Manager     | 01/05/2024    | 500000 |
| 2     | Executive   | 02/05/2024    | 75000  |
| 3     | Manager     | 01/05/2024    | 90000  |
| 2     | Lead        | 02/05/2024    | 85000  |
| 1     | Executive   | 01/05/2024    | 300000 |


_Q1. Write a query to fetch the EmpFname from the EmployeeInfo table in upper case and use the ALIAS name as EmpName._

```sql
SELECT UPPER(EmpFname) AS EmpName
FROM EmployeeInfo;
```

_Q2. Write a query to fetch the number of employees working in the department ‘HR’._

```sql
SELECT COUNT(*) 
FROM EmployeeInfo 
WHERE Department = 'HR';
```

_Q3. Write a query to get the current date._

```sql
SELECT SYSTDATE();
```

_Q4. Write a query to retrieve the first four characters of  EmpLname from the EmployeeInfo table._

```sql
SELECT SUBSTRING(EmpLname, 1, 4) 
FROM EmployeeInfo;
```

_Q5. Write a query to fetch only the place name(string before brackets) from the Address column of EmployeeInfo table._


```sql
SELECT SUBSTRING(Address, 1, CHARINDEX('(', Address))
FROM EmployeeInfo;
```

_Q6. Write a query to create a new table which consists of data and structure copied from the other table._

```sql
CREATE TABLE new_table  AS SELECT * FROM EmployeeInfo;
```

_Q7. Write a query to find all the employees whose salary is between 50000 to 100000._

```sql
SELECT * FROM EmployeePosition
WHERE Salary BETWEEN '50000' AND '100000'
```

_Q8. Write a query to find the names of employees that begin with ‘S’_

```sql
SELECT *
FROM EmployeeInfo
WHERE EmpFname LIKE 'S%';
```

_Q9. Write a query to fetch top N records._

```sql
SELECT * FROM EmployeePosition
ORDER BY Salary DESC 
LIMIT N;
```

_Q10. Write a query to retrieve the EmpFname and EmpLname in a single column as “FullName”. The first name and the last name must be separated with space._

```sql
SELECT CONCAT(EmpFname, ' ', EmpLname) AS 'FullName'
FROM EmployeeInfo;
```

_Q11. Write a query find number of employees whose DOB is between 02/05/1970 to 31/12/1975 and are grouped according to gender_

```sql
SELECT COUNT(*), Gender
FROM EmployeeInfo
WHERE DOB BETWEEN '02/05/1970' AND '31/12/1975';
```

_Q12. Write a query to fetch all the records from the EmployeeInfo table ordered by EmpLname in descending order and Department in the ascending order._

```sql
SELECT * FROM EmployeeInfo 
ORDER BY EmpLname DESC, Department ASC;
```

_Q13. Write a query to fetch details of employees whose EmpLname ends with an alphabet ‘A’ and contains five alphabets._

```sql
SELECT * FROM EmployeeInfo
WHERE EmpLname LIKE '----a';
```

_Q14. Write a query to fetch details of all employees excluding the employees with first names, “Sanjay” and “Sonia” from the EmployeeInfo table._

```sql
SELECT * FROM EmployeeInfo
WHERE EmpFname NOT IN ('Sanjay', 'Sonia');
```

_Q15. Write a query to fetch details of employees with the address as “DELHI(DEL)”._

```sql
SELECT * FROM EmployeeInfo 
WHERE Address LIKE '%DELHI(DEL)%';
```

_Q16. Write a query to fetch all employees who also hold the managerial position._

```sql
SELECT * FROM EmployeeInfo E
INNER JOIN  EmployeePosition P ON E.EmpID = P.EmpID
AND P.EmpPosition IN ('Manager');
```

_Q17. Write a query to fetch the department-wise count of employees sorted by department’s count in ascending order._

```sql
SELECT Department, COUNT(EmpID) as EmpDeptCount
FROM EmployeeInfo
GROUP BY Department
ORDER BY EmpDeptCount;
```

_Q18. Write a query to calculate the even and odd records from a table._

```sql
-- even
SELECT EmpID 
FROM (SELECT rowno, EmpID from EmployeeInfo) 
WHERE MOD(rowno,2)=0;

-- odd
SELECT EmpID FROM (SELECT rowno, EmpID from EmployeeInfo) WHERE MOD(rowno,2)=1;
```

_Q19. Write a SQL query to retrieve employee details from EmployeeInfo table who have a date of joining in the EmployeePosition table._

```sql
SELECT * FROM EmployeeInfo E
WHERE EXISTS
(SELECT * FROM EmployeePosition P WHERE E.EmpID = P.EmpID);
```

_Q20. Write a query to retrieve two minimum and maximum salaries from the EmployeePosition table._

**lowest হইলে E1 বড় হবে, highest হইলে E2 বড় হবে** 

```sql
SELECT DISTINCT Salary                      -- distinct salaries from E1
FROM EmployeePosition E1 
WHERE 2 >= (SELECT COUNT(DISTINCT Salary)   -- number of distinct salaries
            FROM EmployeePosition E2 
            WHERE E1.Salary >= E2.Salary)   -- filters out all but the two lowest salaries
ORDER BY E1.Salary;                         -- ascending order for minimum 
```

1. Outer Query (E1):

Select distinct salaries from the EmployeePosition table.


| Salary |
|--------|
| 45000  |
| 50000  |
| 55000  |
| 70000  |
| 75000  |
| 80000  |

2. Inner Subquery (E2):

For each salary in the outer query, count the number of distinct salaries that are less than or equal to the current salary.

| Salary | Count of Distinct Salaries Less Than or Equal |
|--------|----------------------------------------------|
| 45000  | 1                                            |
| 50000  | 2                                            |
| 55000  | 3                                            |
| 70000  | 4                                            |
| 75000  | 5                                            |
| 80000  | 6                                            |

3. Comparison (WHERE clause):

Filter rows where the count of distinct salaries less than or equal to the current salary is less than or equal to 2.

| Salary |
|--------|
| 45000  |
| 50000  |

4. Result Ordering (ORDER BY):

Order the result set by salary in ascending order.

| Salary |
|--------|
| 45000  |
| 50000  |



```sql
SELECT DISTINCT Salary                      -- distinct salaries from E1
FROM EmployeeInfo E1
WHERE 2 >= (SELECT COUNT(DISTINCT Salary)
            FROM EmployeeInfo E2
            WHERE E1.Salary <= E2.Salary)   -- filters out all but the two highest salaries
ORDER BY E1.Salary DESC;                    -- descending order for maximum
```


2. Inner Subquery (E2):

For each salary in the outer query, count the number of distinct salaries that are greater than or equal to the current salary.

| Salary | Count of Distinct Salaries Greater Than or Equal |
|--------|------------------------------------------------|
| 45000  | 6                                              |
| 50000  | 5                                              |
| 55000  | 4                                              |
| 70000  | 3                                              |
| 75000  | 2                                              |
| 80000  | 1                                              |

3. Comparison (WHERE clause):

Filter rows where the count of distinct salaries greater than or equal to the current salary is less than or equal to 2

| Salary |
|--------|
| 75000  |
| 80000  |

_Q21. Write a query to find the Nth highest salary from the table without using TOP/limit keyword._

**lowest হইলে E1 বড় হবে, highest হইলে E2 বড় হবে** 

```sql
SELECT Salary 
FROM EmployeePosition E1 
WHERE N-1 = (
      -- subquery that calculates the count of distinct salaries greater than the salary of the current row (E1.Salary)
      SELECT COUNT( DISTINCT ( E2.Salary ) ) 
      FROM EmployeePosition E2 
      WHERE E2.Salary > E1.Salary 
);
```

_Q22. Write a query to retrieve duplicate records from a table._


To **retrieve duplicate records** from a table, you can use the SQL **`GROUP BY`** and **`HAVING`** clauses to identify records where there are more than one occurrence

```sql
SELECT EmpID, EmpFname, Department, COUNT(*)
FROM EmployeeInfo
GROUP BY EmpID, EmpFname, Department
HAVING COUNT(*) > 1;
```

_Q23. Write a query to retrieve the list of employees working in the same department._

```sql
SELECT E1.EmpFname, E1.EmpLname, 
       E2.EmpFname, E2.EmpLname
FROM EmployeeInfo E1
INNER JOIN EmployeeInfo E2 ON E1.Department = E2.Department
WHERE E1.EmpID <> E2.EmpID;
```


_Q24. Write a query to retrieve the last 3 records from the EmployeeInfo table._

```sql
SELECT * FROM EmployeeInfo
ORDER BY EmpID DESC
LIMIT 3;
```



_Q25. Write a query to find the third-highest salary from the EmpPosition table._

```sql
SELECT * FROM EmployeePosition
ORDER BY salary DESC
LIMIT 1 OFFSET 2;
```


_Q26. Write a query to display the first and the last record from the EmployeeInfo table._

```sql
SELECT * 
FROM EmployeeInfo 
WHERE EmpID = (SELECT MIN(EmpID) FROM EmployeeInfo)
OR EmpID = (SELECT MAX(EmpID) FROM EmployeeInfo);
```

_Q27. Write a query to add email validation to your database_


In PostgreSQL, you can use the `SIMILAR TO` operator or the `~` operator with the `!~` negation to perform regular expression matching.

```sql
SELECT Email 
FROM EmployeeInfo 
WHERE Email !~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$';
```


_Q28. Write a query to retrieve Departments who have less than 2 employees working in it._

```sql
SELECT Department FROM EmployeeInfo
GROUP BY Department
HAVING COUNT(*) < 2;
```

_Q29. Write a query to retrieve EmpPostion along with total salaries paid for each of them._

```sql
SELECT EmpPosition, SUM(Salary) AS TotalSalary
FROM EmployeePosition
GROUP BY EmpPosition;
```

_Q30. Write a query to fetch 50% records from the EmployeeInfo table._

```sql  
SELECT *
FROM EmployeeInfo
LIMIT (SELECT COUNT(*) * 0.5 FROM EmployeeInfo);
```




### Shiksha 100+ SQL Interview Questions and Answers for 2023

[Shiksha 100+ SQL Interview Questions and Answers for 2023](https://www.shiksha.com/online-courses/articles/top-sql-interview-questions-and-answers/)

_1. Use the GROUP BY clause to **count the number of employees in each department**.mployee table._

```sql
SELECT department, COUNT(EmployeeID) AS num_employee
FROM employee
GROUP BY department;
```

> Note: We should use double quotes or no quotes at all for column aliases in PostgreSQL

_2. Using HAVING clause determine the **department having number of employees greater than 1**._

```sql
SELECT department, COUNT(EmployeeID) AS num_employee
FROM employee
GROUP BY department
HAVING COUNT(EmployeeID) > 1;
```

#### Leet Code SQL Practice Problem

_3. Write a solution to find the customer_number for the **customer who has placed the largest number of orders**._

Orders Table:


| order_number | customer_number |
| -------------|-----------------|
| 1            | 1               |
| 2            | 2               |
| 3            | 3               |
| 4            | 3               |



```SQL
SELECT customer_number 
FROM Orders 
GROUP BY customer_number 
ORDER BY COUNT(customer_number) DESC 
LIMIT 1;
```

_Question-2: Report for every three-line segments whether they can form a triangle. Return the result table in any order._

Triangle table

| Column Name | Type |
|-------------|------|
| x           | int  |
| y           | int  |
| z           | int  |


```sql
SELECT x,y,z
  CASE
    WHEN (x+y) > z AND (y+z) x AND (x+z) > y 
    THEN 'Yes'
    ELSE 'No'
  END AS triangle
FROM triangle;
```

_Question-3: Write a solution to find the names of all the salespersons who did not have any orders related to the company with the name “RED”. Return the result table in any order._


SalesPerson table:

| sales_id | name | salary | commission_rate | hire_date  |
|----------|------|--------|-----------------|------------|
| 1        | John | 100000 | 6               | 4/1/2006   |
| 2        | Amy  | 12000  | 5               | 5/1/2010   |
| 3        | Mark | 65000  | 12              | 12/25/2008 |
| 4        | Pam  | 25000  | 25              | 1/1/2005   |
| 5        | Alex | 5000   | 10              | 2/3/2007   |

Company table:

| com_id | name   | city     |
|--------|--------|----------|
| 1      | RED    | Boston   |
| 2      | ORANGE | New York |
| 3      | YELLOW | Boston   |
| 4      | GREEN  | Austin   |

Orders table:

| order_id | order_date | com_id | sales_id | amount |
|----------|------------|--------|----------|--------|
| 1        | 1/1/2014   | 3      | 4        | 10000  |
| 2        | 2/1/2014   | 4      | 5        | 5000   |
| 3        | 3/1/2014   | 1      | 1        | 50000  |
| 4        | 4/1/2014   | 1      | 4        | 25000  |

Output: 

| name |
|------|
| Amy  |
| Mark |
| Alex |


```sql
SELECT salesperson.name
FROM salesperson
LEFT JOIN orders ON salesperson.sales_id = orders.sales_id
LEFT JOIN company ON orders.com_id = company.com_id
WHERE company.name = 'RED'
GROUP BY salesperson.name
HAVING SUM(amount) IS NULL;
```


Question-4: **Actors and Directors Who Cooperated At Least Three Times** 

_Write a solution to find all the pairs (actor_id, director_id) where the actor has cooperated with the director at least three times. Return the result table in any order._

ActorDirector table:

| actor_id    | director_id | timestamp   |
|-------------|-------------|-------------|
| 1           | 1           | 0           |
| 1           | 1           | 1           |
| 1           | 1           | 2           |
| 1           | 2           | 3           |
| 1           | 2           | 4           |
| 2           | 1           | 5           |
| 2           | 1           | 6           |



| actor_id    | director_id |
|-------------|-------------|
| 1           | 1           |


- **`HAVING` keyword use করতে গেল `GROUP BY` keyword must use করতে হবে** 

```sql
SELECT actor_id, director_id
FROM ActorDirector
GROUP BY actor_id, director_id
HAVING COUNT(timestamp) >= 3;
```

**Question-5: Product Sales Analysis**

_Write a solution to report the product_name, year, and price for each sale_id in the Sales table. Return the resulting table in any order._

Sales table:

| sale_id | product_id | year | quantity | price |
|---------|------------|------|----------|-------| 
| 1       | 100        | 2008 | 10       | 5000  |
| 2       | 100        | 2009 | 12       | 5000  |
| 7       | 200        | 2011 | 15       | 9000  |

Product table:

| product_id | product_name |
|------------|--------------|
| 100        | Nokia        |
| 200        | Apple        |
| 300        | Samsung      |


Output: 

| product_name | year  | price |
|--------------|-------|-------|
| Nokia        | 2008  | 5000  |
| Nokia        | 2009  | 5000  |
| Apple        | 2011  | 9000  |


```sql
SELECT p.product_name, s.year, s.price
FROM sales s
LEFT JOIN product p ON s.product_id = p.product_id;
```


**Question-6: Project Employees**

_Write an SQL query that reports the average experience years of all the employees for each project, rounded to 2 digits. Return the result table in any order._


Project table:

| project_id  | employee_id |
|-------------|-------------|
| 1           | 1           |
| 1           | 2           |
| 1           | 3           |
| 2           | 1           |
| 2           | 4           |


Employee table:

| employee_id | name   | experience_years |
|-------------|--------|------------------|
| 1           | Khaled | 3                |
| 2           | Ali    | 2                |
| 3           | John   | 1                |
| 4           | Doe    | 2                |

Output: 

| project_id  | average_years |
|-------------|---------------|
| 1           | 2.00          |
| 2           | 2.50          |


```sql
SELECT p.project_id, ROUND(AVG(experience_years), 2) AS average_experience_years
FROM project p
JOIN employee ON p.employee_id = e.employee_id
GROUP BY project_id;



-- avg()function use করতে না চাইলে sum() and count() fucntion use করতে হবে 
select project_id,
round(sum(e.experience_years) / count(e.experience_years),2) as average_years
from project p
join employee e on p.employee_id = e.employee_id
group by project_id
```


**Question-7: Game Play Analysis**

_Write a solution to find the first login date for each player. Return the result table in any order._

Activity table:

| player_id | device_id | event_date | games_played |
|-----------|-----------|------------|--------------|
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-05-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |


Output: 

| player_id | first_login |
|-----------|-------------|
| 1         | 2016-03-01  |
| 2         | 2017-06-25  |
| 3         | 2016-03-02  |


```sql
SELECT player_id, MIN(event_date) AS first_login
FROM activity
GROUP BY player_id;
```


**Ques-8: User Activity Past 30 Days**

_Write a solution to find the daily active user count for a period of 30 days ending 2019-07-27 inclusively. A user was active on someday if they made at least one activity on that day. Return the result table in any order._

- 2019 সালের ২৭ জুলাই থেকে তার আগের ৩০ দিন অর্থাৎ জুন মাসের ২৭ তারিখ পর্যন্ত activity count করে query লিখতে হবে (june 27 - july 27). তার আগের activity count হবে না 

Activity table:

| user_id | session_id | activity_date | activity_type |
|---------|------------|---------------|---------------|
| 1       | 1          | 2019-07-20    | open_session  |
| 1       | 1          | 2019-07-20    | scroll_down   |
| 1       | 1          | 2019-07-20    | end_session   |
| 2       | 4          | 2019-07-20    | open_session  |
| 2       | 4          | 2019-07-21    | send_message  |
| 2       | 4          | 2019-07-21    | end_session   |
| 3       | 2          | 2019-07-21    | open_session  |
| 3       | 2          | 2019-07-21    | send_message  |
| 3       | 2          | 2019-07-21    | end_session   |
| 4       | 3          | 2019-06-25    | open_session  |
| 4       | 3          | 2019-06-25    | end_session   |


Output: 

| day        | active_users |
|------------|--------------| 
| 2019-07-20 | 2            |
| 2019-07-21 | 2            |
 

 ```sql
 SELECT activity_date AS day, COUNT( DISTINCT user_id) AS active_users
 FROM activity
 WHERE (activity_date > '2019-06-27' AND activity_date <= '2019-07-27')
 GROUP BY activity_date;
 ```


**Question-9: Article View**

_Write a solution to find all the authors who viewed at least one of their own articles. Return the result table sorted by id in ascending order._


Views table:

| article_id | author_id | viewer_id | view_date  |
|------------|-----------|-----------|------------|
| 1          | 3         | 5         | 2019-08-01 |
| 1          | 3         | 6         | 2019-08-02 |
| 2          | 7         | 7         | 2019-08-01 |
| 2          | 7         | 6         | 2019-08-02 |
| 4          | 7         | 1         | 2019-07-22 |
| 3          | 4         | 4         | 2019-07-21 |
| 3          | 4         | 4         | 2019-07-21 |


Output: 

| id   |
|------|
| 4    |
| 7    |
 

```sql
SELECT DISTINCT(author_id) AS id
FROM views
WHERE author_id = viewer_id
order by id;
```


**Question-10: Average Selling Price**

_Write an SQL query to find the average selling price for each product. average_price should be rounded to 2 decimal places. Return the result table in any order._

Prices table:

| product_id | start_date | end_date   | price  |
|------------|------------|------------|--------|
| 1          | 2019-02-17 | 2019-02-28 | 5      |
| 1          | 2019-03-01 | 2019-03-22 | 20     |
| 2          | 2019-02-01 | 2019-02-20 | 15     |
| 2          | 2019-02-21 | 2019-03-31 | 30     |

UnitsSold table:

| product_id | purchase_date | units |
|------------|---------------|-------|
| 1          | 2019-02-25    | 100   |
| 1          | 2019-03-01    | 15    |
| 2          | 2019-02-10    | 200   |
| 2          | 2019-03-22    | 30    |

Output: 

| product_id | average_price |
|------------|---------------|
| 1          | 6.96          |
| 2          | 16.96         |


Average selling price = Total Price of Product / Number of products sold.

1. Average selling price for product 1 = ((100 * 5) + (15 * 20)) / 115 = 6.96
2. Average selling price for product 2 = ((200 * 15) + (30 * 30)) / 230 = 16.96

```sql
SELECT U.product_id, ROUND(SUM(P.price * U.units) / SUM(U.units),2) AS average_price
FROM prices P
JOIN unitssold U ON 1=1 
AND P.product_id = U.product_id 
AND U.purchase_date BETWEEN P.start_date AND P.end_date
GROUP BY U.product_id;
```


**Question-11: Students and Examination**

Write a sql query solution to find the number of times each student attended each exam. Return the result table ordered by student_id and subject_name.


Students table:

| student_id | student_name |
|------------|--------------|
| 1          | Alice        |
| 2          | Bob          |
| 13         | John         |
| 6          | Alex         |

Subjects table:

| subject_name |
|--------------|
| Math         |
| Physics      |
| Programming  |

Examinations table:

| student_id | subject_name |
|------------|--------------|
| 1          | Math         |
| 1          | Physics      |
| 1          | Programming  |
| 2          | Programming  |
| 1          | Physics      |
| 1          | Math         |
| 13         | Math         |
| 13         | Programming  |
| 13         | Physics      |
| 2          | Math         |
| 1          | Math         |



Output: 

| student_id | student_name | subject_name | attended_exams |
|------------|--------------|--------------|----------------|
| 1          | Alice        | Math         | 3              |
| 1          | Alice        | Physics      | 2              |
| 1          | Alice        | Programming  | 1              |
| 2          | Bob          | Math         | 1              |
| 2          | Bob          | Physics      | 0              |
| 2          | Bob          | Programming  | 1              |
| 6          | Alex         | Math         | 0              |
| 6          | Alex         | Physics      | 0              |
| 6          | Alex         | Programming  | 0              |
| 13         | John         | Math         | 1              |
| 13         | John         | Physics      | 1              |
| 13         | John         | Programming  | 1              |


```sql
SELECT S.student_id, S.student_name, SUB.subject_name, COUNT(E.subject_name) AS attended_exams
FROM students S
LEFT JOIN examinations E ON S.student_id = E.student_id
CROSS JOIN subjects SUB         -- Cartesian product of the S and SUB table
GROUP BY  S.student_id,  S.student_name, SUB.subject_name
ORDER BY S.student_id, SUB.subject_name;
```

**Question-12: Replace Employee ID With The Unique Identifier**

_Write a solution to show the unique ID of each user, If a user does not have a unique ID replace just show null. Return the result table in any order._

Employees table:

| id | name     |
|----|----------|
| 1  | Alice    |
| 7  | Bob      |
| 11 | Meir     |
| 90 | Winston  |
| 3  | Jonathan |

EmployeeUNI table:

| id | unique_id |
|----|-----------|
| 3  | 1         |
| 11 | 2         |
| 90 | 3         |

Output: 

| unique_id | name     |
|-----------|----------|
| null      | Alice    |
| null      | Bob      |
| 2         | Meir     |
| 3         | Winston  |
| 1         | Jonathan |

```sql
SELECT E.name, EU.unique_id AS unique_id
FROM Employees E
LEFT JOIN EmployeeUNI EU ON E.id = EU.id;
```


**Question-13: Top Travellers**

_Write a solution to report the distance traveled by each user. Return the result table ordered by travelled_distance in descending order, if two or more users traveled the same distance, order them by their name in ascending order._

Users table:

| id   | name      |
|------|-----------|
| 1    | Alice     |
| 2    | Bob       |
| 3    | Alex      |
| 4    | Donald    |
| 7    | Lee       |
| 13   | Jonathan  |
| 19   | Elvis     |

Rides table:

| id   | user_id  | distance |
|------|----------|----------|
| 1    | 1        | 120      |
| 2    | 2        | 317      |
| 3    | 3        | 222      |
| 4    | 7        | 100      |
| 5    | 13       | 312      |
| 6    | 19       | 50       |
| 7    | 7        | 120      |
| 8    | 19       | 400      |
| 9    | 7        | 230      |


Output: 

| name     | travelled_distance |
|----------|--------------------|
| Elvis    | 450                |
| Lee      | 450                |
| Bob      | 317                |
| Jonathan | 312                |
| Alex     | 222                |
| Alice    | 120                |
| Donald   | 0                  |



```sql
SELECT U.name, 
    IFNULL(SUM(R.distance), 0) AS travelled_distance
FROM users U 
LEFT JOIN rides R ON U.id = R.user_id
GROUP BY U.id, U.name
ORDER BY travelled_distance DESC, U.NAME ASC;
```

**Question-14: Group Sold Product By Date**

_Write a solution to find for each date the number of different products sold and their names. The sold product names for each date should be sorted lexicographically. Return the result table ordered by sell_date._

Activities table:

| sell_date  | product    |
|------------|------------|
| 2020-05-30 | Headphone  |
| 2020-06-01 | Pencil     |
| 2020-06-02 | Mask       |
| 2020-05-30 | Basketball |
| 2020-06-01 | Bible      |
| 2020-06-02 | Mask       |
| 2020-05-30 | T-Shirt    |

Output: 

| sell_date  | num_sold | products                     |
|------------|----------|------------------------------|
| 2020-05-30 | 3        | Basketball,Headphone,T-shirt |
| 2020-06-01 | 2        | Bible,Pencil                 |
| 2020-06-02 | 1        | Mask                         |


```sql
SELECT DISTINCT(sell_date), COUNT(DISTINCT(product)) AS num_sold
  GROUP_CONCAT(DISTINCT product ORDER BY product ASC SEPERATOR ',') AS products
FROM activities
GROUP BY sell_date
ORDER BY sell_date DESC;
```

**Question-15: Find users with valid E-mail**

Write a solution to find the users who have valid emails.

A valid e-mail has a prefix name and a domain where:

- **The prefix name** is a string that may contain letters (upper or lower case), digits, underscore ‘_’, period ‘.’, and/or dash ‘-‘. The prefix name **must** start with a letter.
- **The domain** is ‘@leetcode.com’.

Users table:

| user_id | name      | mail                    |
|---------|-----------|-------------------------|
| 1       | Winston   | winston@leetcode.com    |
| 2       | Jonathan  | jonathanisgreat         |
| 3       | Annabelle | bella-@leetcode.com     |
| 4       | Sally     | sally.come@leetcode.com |
| 5       | Marwan    | quarz#2020@leetcode.com |
| 6       | David     | david69@gmail.com       |
| 7       | Shapiro   | .shapo@leetcode.com     |

Output: 

| user_id | name      | mail                    |
|---------|-----------|-------------------------|
| 1       | Winston   | winston@leetcode.com    |
| 3       | Annabelle | bella-@leetcode.com     |
| 4       | Sally     | sally.come@leetcode.com |


```sql
SELECT * FROM users
WHERE mail REGEXP '^[a-zA-Z][a-zA-Z0-9_.-]*@leetcode[.]com$';
```


#### Medium
**Question-1: Manager with at least 5 Direct Reports**

Write a solution to find managers with at least five direct reports. Return the result table in any order.

Employee table:

| id  | name  | department | managerId |
|-----|-------|------------|-----------|
| 101 | John  | A          | None      |
| 102 | Dan   | A          | 101       |
| 103 | James | A          | 101       |
| 104 | Amy   | A          | 101       |
| 105 | Anne  | A          | 101       |
| 106 | Ron   | B          | 101       |


Output: 
| name |
|------|
| John |

```sql
SELECT name 
FROM employee AS e 
INNER JOIN employee AS m ON e.managerId = m.Id
GROUP BY e.managerId
HAVING COUNT (e.id) >= 5;
```


**Question-2: Tree Node**

Each node in the tree can be one of three types:

- “Leaf”: if the node is a leaf node.
- “Root”: if the node is the root of the tree.
- “Inner”: If the node is neither a leaf node nor a root node.

Write a solution to report the type of each node in the tree.

![tree1](https://github.com/Mohsem35/DBA/assets/58659448/f1a30364-038c-4ea1-90ad-b6d464cb840e)

Input: 
Tree table:

| id | p_id |
|----|------|
| 1  | null |
| 2  | 1    |
| 3  | 1    |
| 4  | 2    |
| 5  | 2    |

Output: 

| id | type  |
|----|-------|
| 1  | Root  |
| 2  | Inner |
| 3  | Leaf  |
| 4  | Leaf  |
| 5  | Leaf  |

Explanation to Example: 

- Node 1 is the root node because its parent node is null, and it has child nodes 2 and 3.
- Node 2 is an inner node because it has parent node 1 and child nodes 4 and 5.
- Nodes 3, 4, and 5 are leaf nodes because they have parent nodes, and they do not have child nodes.

```sql
SELECT DISTINCT t1.id, (
  CASE 
    WHEN t1.p_id IS NULL THEN 'Root'
    WHEN t1.p_id IS NOT NULL AND t2.p_id IS NOT NULL THEN 'Inner'
    WHEN t1.p_id IS NOT NULL AND t2.p_id IS NULL THEN 'Leaf'  
  END
) AS Type
FROM tree t1
LEFT JOIN tree t2
ON t1.p_id = t2.p_id;
```

**Question-3: Customer Who Bought All Product**

Write a solution to report the customer ids from the Customer table that bought all the products in the Product table. Return the result table in any order.


Customer table:

| customer_id | product_key |
|-------------|-------------|
| 1           | 5           |
| 2           | 6           |
| 3           | 5           |
| 3           | 6           |
| 1           | 6           |

Product table:
| product_key |
|-------------|
| 5           |
| 6           |

Output: 

| customer_id |
|-------------|
| 1           |
| 3           |


```sql
SELECT customer_id FROM customer
GROUP BY customer_id
HAVING COUNT (DISTINCT product_key) = (SELECT COUNT (DISTINCT product_key) 
                                        FROM product);
```
**Question-4: Game Play Analysis**

Write a solution to report the fraction of players that logged in again on the day after the day they first logged in, rounded to 2 decimal places. In other words, you need to count the number of players that logged in for at least two consecutive days starting from their first login date, then divide that number by the total number of players.


Activity table:

| player_id | device_id | event_date | games_played |
|-----------|-----------|------------|--------------|
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-03-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |

Output: 

| fraction  |
|-----------|
| 0.33      |


Total player হচ্ছে ৩ জন, শুধুমাত্র 1 no player login করার ঠিক পরের দিন আবার ঢুকে game play করে। তাহলে result হচ্ছে 1/3 = 0.33

```sql
WITH Firstlogins AS (
  SELECT player_id, MIN(event_date) AS first_login
  FROM Activity
  GROUP BY player_id
),

ConsecutiveLogins AS (
  SELECT f1.player_id
  FROM Firstlogins f1
  INNER JOIN Activity act ON f1.player_id = act.player_id
  -- main line 
  WHERE act.event_date = DATE_ADD(f1.first_login, INTERVAL 1 DAY)
)

SELECT CAST (COUNT(*) AS FLOAT) / COUNT(DISTINCT player_id) AS fraction
FROM ConsecutiveLogins;
```


**Question-5: Product Price At a Given Date**

বুঝি নাই problem টা 

Write an SQL query to find the prices of all products on 2019-08-16. Assume the price of all products before any change is 10. Return the result table in any order


Products table:

| product_id | new_price | change_date |
|------------|-----------|-------------|
| 1          | 20        | 2019-08-14  |
| 2          | 50        | 2019-08-14  |
| 1          | 30        | 2019-08-15  |
| 1          | 35        | 2019-08-16  |
| 2          | 65        | 2019-08-17  |
| 3          | 20        | 2019-08-18  |

Output: 

| product_id | price |
|------------|-------|
| 2          | 50    |
| 1          | 35    |
| 3          | 10    |


```sql
SELECT p1.product_id,
  COALESCE(MAX(CASE 
                 WHEN change_date <= '2019-08-16' THEN new_price 
                 ELSE NULL 
               END), 10) AS price
FROM Products p1
LEFT JOIN Products p2 ON p1.product_id = p2.product_id
GROUP BY p1.product_id;

```

`COALESCE(MAX(CASE WHEN change_date <= '2019-08-16' THEN new_price ELSE NULL END), 10) AS price`

এই অংশ একটি নতুন দাম ফিল্ড তৈরি করবে যেটি সর্বোচ্চ দাম নিবে কোন প্রোডাক্টের জন্য। যদি change_date '2019-08-16' এর আগে হয়, তাহলে সেই প্রোডাক্টের নতুন দাম নিবে, অন্যথায় সে ১০ টাকা হিসেবে ফিরে দেবে। COALESCE ফাংশনটি সর্বাধিক দাম ফিল্ডের মান নিবে কিন্তু যদি এক্সপ্রেশনটি NULL হয়, তাহলে সে বিকল্প হিসেবে ১০ টাকা নিবে।


**Question-6: Monthly Transaction**

বুঝি নাই 

Write an SQL query to find for each month and country, the number of transactions and their total amount, the number of approved transactions and their total amount. Return the result table in any order.


Transactions table:

| id   | country | state    | amount | trans_date |
|------|---------|----------|--------|------------|
| 121  | US      | approved | 1000   | 2018-12-18 |
| 122  | US      | declined | 2000   | 2018-12-19 |
| 123  | US      | approved | 2000   | 2019-01-01 |
| 124  | DE      | approved | 2000   | 2019-01-07 |


Output: 

| month    | country | trans_count | approved_count | trans_total_amount | approved_total_amount |
|----------|---------|-------------|----------------|--------------------|-----------------------|
| 2018-12  | US      | 2           | 1              | 3000               | 1000                  |
| 2019-01  | US      | 1           | 1              | 2000               | 2000                  |
| 2019-01  | DE      | 1           | 1              | 2000               | 2000                  |



```sql
SELECT
  YEAR(trans_date) AS year,
  MONTH(trans_date) AS month,
  country,
  COUNT(*) AS trans_count,
  SUM(amount) AS trans_total_amount,
  SUM(CASE WHEN state = 'approved' THEN amount ELSE 0 END) AS approved_count,
  SUM(CASE WHEN state = 'approved' THEN amount ELSE 0 END) AS approved_total_amount
FROM Transactions
GROUP BY YEAR(trans_date), MONTH(trans_date), country
ORDER BY year, month, country;
```

**Question-7: Last Person to fit in Bus**

There is a queue of people waiting to board a bus. However, the bus has a weight limit of  1000 kilograms, so there may be some people who cannot board.

Write a solution to find the person_name of the last person that can fit on the bus without exceeding the weight limit. The test cases are generated such that the first person does not exceed the weight limit.


Queue table:

| person_id | person_name | weight | turn |
|-----------|-------------|--------|------|
| 5         | Alice       | 250    | 1    |
| 4         | Bob         | 175    | 5    |
| 3         | Alex        | 350    | 2    |
| 6         | John Cena   | 400    | 3    |
| 1         | Winston     | 500    | 6    |
| 2         | Marie       | 200    | 4    |

Output: 

| person_name |
|-------------|
| John Cena   |

```sql

WITH CumulativeWeight AS (
  SELECT person_id, person_name, weight, turn,
         SUM(weight) OVER (ORDER BY turn) AS cumulative_weight
  FROM Queue
)
SELECT person_name
FROM CumulativeWeight
WHERE cumulative_weight <= 1000
ORDER BY turn DESC
LIMIT 1;
```
The CTE calculates the cumulative weight for each person based on their position in the queue (turn). The main query then finds the first person (ordered by descending turn) whose cumulative weight is less than or equal to the weight limit. This ensures we identify the last person who can board without exceeding the limit


**Question-8: Restaurant Growth**

You are the restaurant owner and want to analyze a possible expansion (there will be at least one customer daily). Compute the moving average of how much the customer paid in a seven-day window (i.e., current day + 6 days before). average_amount should be rounded to two decimal places.

Return the result table ordered by visited_on in ascending order.

Customer table:

| customer_id | name         | visited_on   | amount      |
|-------------|--------------|--------------|-------------|
| 1           | Jhon         | 2019-01-01   | 100         |
| 2           | Daniel       | 2019-01-02   | 110         |
| 3           | Jade         | 2019-01-03   | 120         |
| 4           | Khaled       | 2019-01-04   | 130         |
| 5           | Winston      | 2019-01-05   | 110         | 
| 6           | Elvis        | 2019-01-06   | 140         | 
| 7           | Anna         | 2019-01-07   | 150         |
| 8           | Maria        | 2019-01-08   | 80          |
| 9           | Jaze         | 2019-01-09   | 110         | 
| 1           | Jhon         | 2019-01-10   | 130         | 
| 3           | Jade         | 2019-01-10   | 150         | 

Output: 

You are the restaurant owner and want to analyze a possible expansion (there will be at least one customer daily). Compute the moving average of how much the customer paid in a seven-day window (i.e., current day + 6 days before). average_amount should be rounded to two decimal places.

Return the result table ordered by visited_on in ascending order.

Customer table:

| customer_id | name         | visited_on   | amount      |
|-------------|--------------|--------------|-------------|
| 1           | Jhon         | 2019-01-01   | 100         |
| 2           | Daniel       | 2019-01-02   | 110         |
| 3           | Jade         | 2019-01-03   | 120         |
| 4           | Khaled       | 2019-01-04   | 130         |
| 5           | Winston      | 2019-01-05   | 110         | 
| 6           | Elvis        | 2019-01-06   | 140         | 
| 7           | Anna         | 2019-01-07   | 150         |
| 8           | Maria        | 2019-01-08   | 80          |
| 9           | Jaze         | 2019-01-09   | 110         | 
| 1           | Jhon         | 2019-01-10   | 130         | 
| 3           | Jade         | 2019-01-10   | 150         | 

Output: 

| visited_on   | amount       | average_amount |
|--------------|--------------|----------------|
| 2019-01-07   | 860          | 122.86         |
| 2019-01-08   | 840          | 120            |
| 2019-01-09   | 840          | 120            |
| 2019-01-10   | 1000         | 142.86         |


```sql
WITH CumulativeSum AS (
  SELECT
    customer_id,
    name,
    visited_on,
    amount,
    SUM(amount) OVER (PARTITION BY visited_on ORDER BY visited_on ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS seven_day_sum
  FROM Customer
)
SELECT
  visited_on,
  amount,
  ROUND(CAST(seven_day_sum AS DECIMAL(10,2)) / 7, 2) AS average_amount
FROM CumulativeSum
WHERE seven_day_sum IS NOT NULL -- Filter for rows with a calculated seven_day_sum
ORDER BY visited_on;
```

**Question-9: Movie Rating**

Write a solution to:

- Find the user’s name who has rated the greatest number of movies. In case of a tie, return the lexicographically smaller user name.

- Find the movie name with the highest average rating in February 2020. In case of a tie, return the lexicographically smaller movie name.

Movies table:
| movie_id    |  title       |
|-------------|--------------|
| 1           | Avengers     |
| 2           | Frozen 2     |
| 3           | Joker        |

Users table:
| user_id     |  name        |
|-------------|--------------|
| 1           | Daniel       |
| 2           | Monica       |
| 3           | Maria        |
| 4           | James        |

MovieRating table:
| movie_id    | user_id      | rating       | created_at  |
|-------------|--------------|--------------|-------------|
| 1           | 1            | 3            | 2020-01-12  |
| 1           | 2            | 4            | 2020-02-11  |
| 1           | 3            | 2            | 2020-02-12  |
| 1           | 4            | 1            | 2020-01-01  |
| 2           | 1            | 5            | 2020-02-17  | 
| 2           | 2            | 2            | 2020-02-01  | 
| 2           | 3            | 2            | 2020-03-01  |
| 3           | 1            | 3            | 2020-02-22  | 
| 3           | 2            | 4            | 2020-02-25  | 

Output: 

| results      |
|--------------|
| Daniel       |
| Frozen 2     |

```sql
-- Find user with most movie ratings
WITH UserRatingCount AS (
  SELECT u.name, COUNT(*) AS rating_count
  FROM MovieRating mr
  JOIN Users u ON mr.user_id = u.user_id
  GROUP BY u.name
  ORDER BY rating_count DESC, name ASC
  LIMIT 1
)
SELECT results
FROM UserRatingCount;

-- Find movie with highest average rating in February 2020
WITH MovieAvgRating AS (
  SELECT m.title, AVG(rating) AS avg_rating
  FROM MovieRating mr
  JOIN Movies m ON mr.movie_id = m.movie_id
  WHERE MONTH(created_at) = 2 AND YEAR(created_at) = 2020
  GROUP BY m.title
  ORDER BY avg_rating DESC, title ASC
  LIMIT 1
)
SELECT results
FROM MovieAvgRating;
```

**Question-10: Capital Gain/Loss**

Write a solution to report the Capital gain/loss for each stock. The Capital gain/loss of a stock is the total gain or loss after buying and selling the stock one or many times.

Return the result table in any order.

Stocks table:

| stock_name    | operation | operation_day | price  |
|---------------|-----------|---------------|--------|
| Leetcode      | Buy       | 1             | 1000   |
| Corona Masks  | Buy       | 2             | 10     |
| Leetcode      | Sell      | 5             | 9000   |
| Handbags      | Buy       | 17            | 30000  |
| Corona Masks  | Sell      | 3             | 1010   |
| Corona Masks  | Buy       | 4             | 1000   |
| Corona Masks  | Sell      | 5             | 500    |
| Corona Masks  | Buy       | 6             | 1000   |
| Handbags      | Sell      | 29            | 7000   |
| Corona Masks  | Sell      | 10            | 10000  |

Output: 

| stock_name    | capital_gain_loss |
|---------------|-------------------|
| Corona Masks  | 9500              |
| Leetcode      | 8000              |
| Handbags      | -23000            |


```sql
SELECT stock_name, SUM(
    CASE
        WHEN operation = 'Buy' THEN -price
        ELSE price
    END
) AS capital_gain_loss
FROM Stocks
GROUP BY stock_name
```


#### FAANG QUESTION FOR PRACTICE**

**1. Facebook – Pages with No Likes**

Assume you’re given two tables containing data about Facebook Pages and their respective likes (as in “Like a Facebook Page”). Write a query to return the IDs of the Facebook pages with zero likes. The output should be sorted in ascending order based on the page IDs.\

`pages` table:


| page_id   | page_name |
|-----------|-------------------|
| 20001 | SQL Solutions |
| 20045 | Brain Exercises |
| 20701	| Tips for Data Analysts |

`page_likes` table

| user_id | page_id | liked_date         |
|---------|---------|--------------------|
| 111     | 20001   | 04/08/2022 00:00:00|
| 121     | 20045   | 03/12/2022 00:00:00|
| 156     | 20001   | 07/25/2022 00:00:00|

| page_id |
|---------|
| 20701   |


```sql
SELECT p.page_id FROM pages p
LEFT JOIN page_likes pl ON p.page_id = pl.page_id
-- 2 টা table join করার পরে যেইটা null থাকব সেইটাই like পায় নাই 
WHERE pl.page_id IS NULL
ORDER BY pl.page_id
```

**2. Twitter – Histogram of Tweets**

Assume you’re given a table of Twitter tweet data and write a query to obtain a histogram of tweets posted per user in 2022. Output the tweet counts per user as the bucket and the number of Twitter users who fall into that bucket. In other words, group the users by the number of tweets they posted in 2022 and count the number of users in each group.

`tweets` table input:

| tweet_id | user_id | msg                                                        | tweet_date           |
|----------|---------|------------------------------------------------------------|----------------------|
| 214252   | 111     | Am considering taking Tesla private at $420. Funding secured. | 12/30/2021 00:00:00 |
| 739252   | 111     | Despite the constant negative press covfefe                 | 01/01/2022 00:00:00 |
| 846402   | 111     | Following @NickSinghTech on Twitter changed my life!        | 02/14/2022 00:00:00 |
| 241425   | 254     | If the salary is so competitive why won’t you tell me what it is? | 03/01/2022 00:00:00 |
| 231574   | 148     | I no longer have a manager. I can't be managed             | 03/23/2022 00:00:00 |


Output:

| tweet_bucket | users_num |
|--------------|-----------|
| 1            | 2         |
| 2            | 1         |

**Explanation:**

Based on the example output, there are two users who posted only one tweet in 2022, and one user who posted two tweets in 2022. The query groups the users by the number of tweets they posted and displays the number of users in each group.


```sql
SELECT tweet_count_per_user AS tweet_bucket, COUNT(user_id) AS users_num
FROM (
  SELECT user_id, COUNT(tweet_id) AS tweet_count_per_user
  FROM tweets
  WHERE YEAR(tweet_date) = 2022
  GROUP BY user_id
) AS total_tweets
GROUP BY tweet_count_per_user;
```

**3. Facebook – App Click-through Rate (CTR)**
Assume you have an events table on Facebook app analytics. Write a query to calculate the click-through rate (CTR) for the app in 2022 and round the results to 2 decimal places.

Definition and note:

- Percentage of click-through rate (CTR) = 100.0 * Number of clicks / Number of impressions
- To avoid integer division, multiply the CTR by 100.0, not 100.

`events` table

| app_id | event_type | timestamp           |
|--------|------------|---------------------|
| 123    | impression | 07/18/2022 11:36:12 |
| 123    | impression | 07/18/2022 11:37:12 |
| 123    | click      | 07/18/2022 11:37:42 |
| 234    | impression | 07/18/2022 14:15:12 |
| 234    | click      | 07/18/2022 14:16:12 |


Output:

| app_id | ctr    |
|--------|--------|
| 123    | 50.00  |
| 234    | 100.00 |

**Explanation**

Let's consider an example of App 123. This app has a click-through rate (CTR) of 50.00% because out of the 2 impressions it received, it got 1 click.

To calculate the CTR, we divide the number of clicks by the number of impressions, and then multiply the result by 100.0 to express it as a percentage. In this case, 1 divided by 2 equals 0.5, and when multiplied by 100.0, it becomes 50.00%. So, the CTR of App 123 is 50.00%.

```sql
SELECT 
    app_id, 
    ROUND(
        100.0 * SUM(CASE WHEN event_type = 'click' THEN 1 ELSE 0 END) / 
        SUM(CASE WHEN event_type = 'impression' THEN 1 ELSE 0 END), 
    2) AS ctr
FROM events
WHERE YEAR(timestamp) = 2022
GROUP BY app_id;
```

**4. New York Times – Laptop vs Mobile Viewership**

Assume you’re given the table on user viewership categorized by device type, where the three types are laptop, tablet, and phone. Write a query that calculates the total viewership for laptops and mobile devices, where mobile is defined as the sum of tablet and phone viewership. Output the total viewership for laptops as laptop_reviews and the total viewership for mobile devices as mobile_views.

`viewership` table

| user_id | device_type | view_time           |
|---------|-------------|---------------------|
| 123     | tablet      | 01/02/2022 00:00:00 |
| 125     | laptop      | 01/07/2022 00:00:00 |
| 128     | laptop      | 02/09/2022 00:00:00 |
| 129     | phone       | 02/09/2022 00:00:00 |
| 145     | tablet      | 02/24/2022 00:00:00 |

Output

| laptop_views | mobile_views |
|--------------|--------------|
| 2            | 3            |

```sql
SELECT 
SUM(CASE WHEN device_type='laptop' THEN 1 ELSE 0 END) AS laptop_reviews,
SUM(CASE WHEN device_type IN ('phone','tablet') THEN 1 ELSE 0 END) AS mobile_reviews
FROM viewership;
```

**5. Microsoft – Teams Power Users**

Write a query to identify the top 2 Power Users who sent the most messages on Microsoft Teams in August 2022. Display these 2 users’ IDs and the total number of messages they sent. Output the results in descending order based on the count of the messages.

Assumption: No two users have sent the same number of messages in August 2022.

`messages` table

| message_id | sender_id | receiver_id | content                  | sent_date           |
|------------|-----------|-------------|--------------------------|---------------------|
| 901        | 3601      | 4500        | You up?                  | 08/03/2022 00:00:00 |
| 902        | 4500      | 3601        | Only if you're buying    | 08/03/2022 00:00:00 |
| 743        | 3601      | 8752        | Let's take this offline | 06/14/2022 00:00:00 |
| 922        | 3601      | 4500        | Get on the call          | 08/10/2022 00:00:00 |

Output:

| sender_id | message_count |
|-----------|---------------|
| 3601      | 2             |
| 4500      | 1             |

```sql
SELECT sender_id, COUNT(message_id) AS message_count
FROM messages
WHERE EXTRACT(MONTH FROM sent_date) = 8 AND EXTRACT(YEAR FROM sent_date) = 2022
GROUP BY sender_id
ORDER BY message_count DESC
LIMIT 2;
```

**6. Linkedin – Duplicate Job Listings**

Assume you’re given a table containing job postings from various companies on the LinkedIn platform. Write a query to retrieve the count of companies that have posted duplicate job listings.

Definition:
- Duplicate job listings are defined as two job listings within the same company that share identical titles and descriptions.

`job_listings` table:

| job_id | company_id | title           | description                                                                                                                                                                     |
|--------|------------|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 248    | 827        | Business Analyst | Business analyst evaluates past and current business data with the primary goal of improving decision-making processes within organizations.                                   |
| 149    | 845        | Business Analyst | Business analyst evaluates past and current business data with the primary goal of improving decision-making processes within organizations.                                   |
| 945    | 345        | Data Analyst    | Data analyst reviews data to identify key insights into a business's customers and ways the data can be used to solve problems.                                                 |
| 164    | 345        | Data Analyst    | Data analyst reviews data to identify key insights into a business's customers and ways the data can be used to solve problems.                                                 |
| 172    | 244        | Data Engineer   | Data engineer works in a variety of settings to build systems that collect, manage, and convert raw data into usable information for data scientists and business analysts to interpret. |

Output:

| duplicate_companies |
|---------------------|
| 1                   |

**Explanation:**

There is one company ID 345 that posted duplicate job listings. The duplicate listings, IDs 945 and 164 have identical titles and descriptions.

```sql
SELECT COUNT(DISTINCT company_id) AS duplicate_companies
FROM (
  SELECT company_id, title, description
  FROM job_listings
  GROUP BY company_id, title, description
  HAVING COUNT(*) > 1
) AS duplicates;
```


**7. Amazon – Average Review Rating**

Given the reviews table, write a query to retrieve the average star rating for each product, grouped by month. The output should display the month as a numerical value, product ID, and average star rating rounded to two decimals. Sort the output first by month and then by product ID.

`reviews` table

| review_id | user_id | submit_date         | product_id | stars |
|-----------|---------|---------------------|------------|-------|
| 6171      | 123     | 06/08/2022 00:00:00 | 50001      | 4     |
| 7802      | 265     | 06/10/2022 00:00:00 | 69852      | 4     |
| 5293      | 362     | 06/18/2022 00:00:00 | 50001      | 3     |
| 6352      | 192     | 07/26/2022 00:00:00 | 69852      | 3     |
| 4517      | 981     | 07/05/2022 00:00:00 | 69852      | 2     |

Output:

| mth | product | avg_stars |
|-----|---------|-----------|
| 6   | 50001   | 3.50      |
| 6   | 69852   | 4.00      |
| 7   | 69852   | 2.50      |


**Explanation**

Product 50001 received two ratings of 4 and 3 in the month of June (6th month), resulting in an average star rating of 3.5.

```sql
SELECT EXTRACT(MONTH FROM submit_date) AS mth,
       product_id,
       ROUND(AVG(stars), 2) AS avg_stars
FROM reviews
GROUP BY EXTRACT(MONTH FROM submit_date), product_id
ORDER BY mth, product_id;
```

**8. JPMorgan – Cards Issued Difference**

Your team at JPMorgan Chase is preparing to launch a new credit card, and to gain some insights, you’re analyzing how many credit cards were issued each month.

Write a query that outputs the name of each credit card and the difference in the number of issued cards between the month with the highest and lowest issuance cards. Arrange the results based on the largest disparity.

`monthly_cards_issued` table

| card_name            | issued_amount | issue_month | issue_year |
|----------------------|---------------|-------------|------------|
| Chase Freedom Flex   | 55000         | 1           | 2021       |
| Chase Freedom Flex   | 60000         | 2           | 2021       |
| Chase Freedom Flex   | 65000         | 3           | 2021       |
| Chase Freedom Flex   | 70000         | 4           | 2021       |
| Chase Sapphire Reserve | 170000      | 1           | 2021       |
| Chase Sapphire Reserve | 175000      | 2           | 2021       |
| Chase Sapphire Reserve | 180000      | 3           | 2021       |

Output:

| card_name            | difference |
|----------------------|------------|
| Chase Freedom Flex   | 15000      |
| Chase Sapphire Reserve | 10000    |

```sql
WITH highest_monthly_issued AS (
  SELECT card_name, MAX(issued_amount) AS highest_issued
  FROM monthly_cards_issued
  GROUP BY card_name
),
lowest_monthly_issued AS (
  SELECT card_name, MIN(issued_amount) AS lowest_issued
  FROM monthly_cards_issued
  GROUP BY card_name
)
SELECT hmi.card_name, hmi.highest_issued - lmi.lowest_issued AS difference
FROM highest_monthly_issued AS hmi
JOIN lowest_monthly_issued AS lmi ON hmi.card_name = lmi.card_name
ORDER BY difference DESC;
```

**9. Cities With Completed Trades**

Assume you’re given the tables containing completed trade orders and user details in a Robinhood trading system.

Write a query to retrieve the top three cities that have the highest number of completed trade orders listed in descending order. Output the city name and the corresponding number of completed trade orders.

`trades` table

| order_id | user_id | price | quantity | status    | timestamp           |
|----------|---------|-------|----------|-----------|---------------------|
| 100101   | 111     | 9.80  | 10       | Cancelled | 08/17/2022 12:00:00 |
| 100102   | 111     | 10.00 | 10       | Completed | 08/17/2022 12:00:00 |
| 100259   | 148     | 5.10  | 35       | Completed | 08/25/2022 12:00:00 |
| 100264   | 148     | 4.80  | 40       | Completed | 08/26/2022 12:00:00 |
| 100305   | 300     | 10.00 | 15       | Completed | 09/05/2022 12:00:00 |
| 100400   | 178     | 9.90  | 15       | Completed | 09/09/2022 12:00:00 |
| 100565   | 265     | 25.60 | 5        | Completed | 12/19/2022 12:00:00 |

`users` table


| user_id | city          | email                              | signup_date           |
|---------|---------------|------------------------------------|-----------------------|
| 111     | San Francisco | rrok10@gmail.com                  | 08/03/2021 12:00:00  |
| 148     | Boston        | sailor9820@gmail.com              | 08/20/2021 12:00:00  |
| 178     | San Francisco | harrypotterfan182@gmail.com       | 01/05/2022 12:00:00  |
| 265     | Denver        | shadower_@hotmail.com             | 02/26/2022 12:00:00  |
| 300     | San Francisco | houstoncowboy1122@hotmail.com     | 06/30/2022 12:00:00  |

Output

| city          | total_orders |
|---------------|--------------|
| San Francisco | 3            |
| Boston        | 2            |
| Denver        | 1            |


**Explanation**

In the given dataset, San Francisco has the highest number of completed trade orders with 3 orders. Boston holds the second position with 2 orders, and Denver ranks third with 1 order.

```sql
SELECT u.city, COUNT(t.order_id) as total_orders
  FROM trades t
  LEFT JOIN users u ON t.user_id = u.user_id
  WHERE t.status = 'Completed'
GROUP BY city
ORDER BY total_orders DESC
LIMIT 3
```
**10. Alibaba – Compressed Mean**

You're trying to find the mean number of items per order on Alibaba, rounded to 1 decimal place using tables which includes information on the count of items in each order (item_count table) and the corresponding number of orders for each item count (order_occurrences table).

`items_per_order` table

| item_count | order_occurrences |
|------------|-------------------|
| 1          | 500               |
| 2          | 1000              |
| 3          | 800               |
| 4          | 1000              |

| mean |
|------|
| 2.7  |

**Explanation**
Let's calculate the arithmetic average:

Total items = (1*500) + (2*1000) + (3*800) + (4*1000) = 8900

Total orders = 500 + 1000 + 800 + 1000 = 3300

Mean = 8900 / 3300 = 2.7

```sql
SELECT ROUND(AVG(item_count * order_occurrences) / SUM(order_occurrences), 1) AS mean
FROM items_per_order;
```