
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
SELECT DISTINCT(sell_date), COUNT(DISTINCT(product)) AS num_sold, 
```