
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