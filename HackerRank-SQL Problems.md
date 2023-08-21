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
