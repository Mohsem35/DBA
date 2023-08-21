## HackerRank-SQL(Basic)SkillsCertificationTest

Question 1: A university has created a rule that every student will have their own supervisor. **`Write a query that will give a student list with name and roll_number where either advisor is Male with 
salary greater than 15000 or Female advisor with salary greater than 20000`**. There are two tables: student_information and faculty_information 

```
SELECT roll_number, name
FROM student_information
WHERE advisor IN (
    SELECT employee_id
    FROM faculty_information
    WHERE (gender = M AND salary > 15000)
       OR (gender = F AND salary > 20000)
);
```

Question 2: A school recently conducted its annual examination and wishes to know the list of academically low-performing students to organize extra classes for them. 
**`Write a query to return the roll number and names of students who have a total of less than 100 marks including all 3 subjects`**. There are two tables: student_information and examination_marks. 
Their primary keys are roll_number.

```
SELECT si.roll_number, si.name
FROM student_information si
JOIN (
    SELECT roll_number, (subject_one + subject_two + subject_three) AS total_sum
    FROM examination_marks
) em ON si.roll_number = em.roll_number
WHERE em.total_sum < 100;
```

## HackerRank Problem Solving

#### Revising the Select Query I
Query all columns for all American cities in the CITY table with populations larger than 100000. The CountryCode for America is USA.

```
SELECT *
FROM city
WHERE countrycode = 'USA' AND population > 100000;
```

#### Revising the Select Query II
Query the NAME field for all American cities in the CITY table with populations larger than 120000. The CountryCode for America is USA.

```
SELECT NAME
FROM city
WHERE countrycode = 'USA' AND population > 120000;
```

#### Select All
Query all columns (attributes) for every row in the CITY table.

```
SELECT * FROM CITY;
```

#### Japanese Cities' Attributes

Query all attributes of every Japanese city in the CITY table. The COUNTRYCODE for Japan is JPN.



```
SELECT * FROM city WHERE countrycode = 'JPN';
```


#### Japanese Cities' Names

```
SELECT name FROM city WHERE countrycode = 'JPN';
```


#### Weather Observation Station 1

```
SELECT city,state FROM station;
```

#### Weather Observation Station 3

Query a list of CITY names from STATION for cities that have an even ID number. Print the results in any order, but exclude duplicates from the answer.
```
SELECT DISTINCT CITY
FROM STATION
WHERE MOD(ID,2) = 0;
```
