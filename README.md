# Language-Center-DB-SQL-Project
This SQL project is based on an academic task focused on database design and manipulation. It involves creating and managing a database for a language training centre, including tables for Teachers, Staff Registrations, Courses, Students, and Student Enrolments.

- Benefits: This database supports efficient management of teacher assignments, student enrolments, course offerings, and staff registrations. SQL manipulation ensures optimised queries, data integrity, and streamlined reporting.
- The query to create the database can be viewed at [Create_database_and_tables](https://github.com/Yuxi-Cn/Language-Center-DB-SQL-Project/blob/main/Create_database_and_tables).
- The GPT-generated (dummy) data and insert queries can be viewed at [link].

## Project Introduction
While the original project required only the database design, I was interested in exploring further manipulation possibilities within this database. This repository demonstrates some hypothesised scenarios with solutions using MySQL Workbench, aimed at simplifying reporting and enabling quick data retrieval.

### ER-Diagram
The Entity Relationship Diagram is shown below. You can also find a detailed explanation of the relationships here: [link].

![Language-Center_ER-Diagram](https://github.com/user-attachments/assets/6938d3f2-409f-4368-bc90-0516bf946139)

### Assumed Scenarios
Four scenarios are assumed with correpsonding requirement for SQL commands.

| Scenario | Description                                                                                             |
|----------|---------------------------------------------------------------------------------------------------------|
| S1       | The training center doesn't allow late enrollment since 2023, so we need to find students who enrolled after the deadline and between 2023-06-20 and 2023-09-10.                  |
| S2       | The center wants to recognise the best Science teachers, and we should find the top 3 Science teachers with the highest ratings in 2022.                                          |
| S3       | The center plans to award scholarships, it's required to find the top 10 students with the highest average course scores.                                                          |
| S4       | The center wants to honor the best students per department, we need to find the students with the highest total scores in each department.                                         |

## SQL Solutions
To meet the requirements listed above, let's dive into our solutions for each scenario!

### 1. Find students who enrolled after the enrolment deadline and where the enrollment date is between 2023-06-20 and 2023-09-10.

We need to filter student enrolment dates based on two conditions:

- The `enrollment_date` is later than the `enrollment_deadline`.
- The `enrollment_date` falls within the period from 2023-06-20 to 2023-09-10.

**Step 1:** Join the *Enrollment* table and the *Students* table to obtain student names along with their enrolment information.

**Step 2:** Apply the conditions in the `WHERE` clause to filter the students accordingly.

```
SELECT
      s.first_name, 
      s.surname, 
      e.reference_number, 
      e.enrollment_date, 
      e.enrollment_deadline
FROM Enrollment e
JOIN Student s USING(student_id)
WHERE enrollment_date > enrollment_deadline
  AND enrollment_date BETWEEN '2023-06-20' AND '2023-09-10';
```

### 2. Find the top 3 Science teachers with the highest ratings in 2022.

To find the names of the top 3 Science teachers with the highest rating scores in 2022, we need to apply the following criteria:

- The rating year must be 2022.
- Only the top 3 teacher names should be returned.

**Step 1:** Use the `CONCAT()` function to combine the teacher's first name and last name, with a space between them.

**Step 2:** Filter the teachers by using the `YEAR()` function in the `WHERE` clause to ensure the ratings are from 2022.

**Step 3:** Use `ORDER BY` with `DESC` to sort the results in descending order of ratings, and set `LIMIT` to 3 to return only the top 3 teachers.

```
SELECT
      employee_id, 
      CONCAT(first_name, ' ', surname) AS teacher_name,
      teacher_rating
FROM Teacher
WHERE YEAR(rate_date) = 2022
ORDER BY teacher_rating DESC
LIMIT 3;
```

### 3. Find the top 10 students with the highest average course scores.

To find the top 10 students with the highest average course scores, we need to consider the following:

- Course scores should be aggregated as an average.
- Only the top 10 students with the highest average scores should be returned.

**Step 1:** Use the `CONCAT()` function to combine the student's first name and last name with a space in between.

**Step 2:** Calculate the average scores and limit the decimal places to 2 using the `AVG()` and `ROUND()` functions.

**Step 3:** Join tables:
- Use `LEFT JOIN` to join the *Students* table with the *Studentcourse* table using `student_id`.
- Use another `LEFT JOIN` to join the *Studentcourse* table with the *Course* table using `course_id`.

**Step 4:** Apply `GROUP BY` on `student_id` to ensure the `course_score` is aggregated correctly.

**Step 5:** Use `ORDER BY` with `DESC` to sort the results in descending order of average scores, and set `LIMIT` to 10 to return only the top 10 students.

```
SELECT 
      s.student_id,
      CONCAT(s.first_name, ' ', s.surname) AS student_name,
      ROUND(AVG(c.course_score), 1) AS average_score
FROM Student s
LEFT JOIN Studentcourse sc ON s.student_id = sc.student_id
LEFT JOIN Course c ON sc.course_id = c.course_id
GROUP BY s.student_id
ORDER BY AVG(c.course_score) DESC
LIMIT 10;
```
> [!TIP]  
> If the scores do not need to be returned, the `RANK() OVER()` function can be used to directly determine the top 10 students by ranking their average scores in descending order. Apart from using a *common table expression (CTE)* and limiting the rank to 10 or less, the rest of the process basically remains the same.

```
WITH Avg_score_rank AS (
  SELECT 
        s.student_id,
        CONCAT(s.first_name, ' ', s.surname) AS student_name,
        RANK() OVER (ORDER BY AVG(c.course_score) DESC) AS rk
  FROM Student s
  LEFT JOIN Studentcourse sc ON s.student_id = sc.student_id
  LEFT JOIN Course c ON sc.course_id = c.course_id
  GROUP BY student_id
)
SELECT * FROM Avg_score_rank
WHERE rk < 11;
```

### 4. Find the best students with the highest total score in each department.

Q4 requires finding the top student with the highest total score in every department. The criteria are:

- The course scores should be aggregated as a total.
- Only the student names with the highest total score should be returned.

**Step 1:** Create a CTE named *total_score_list* to return the student information (ID and full names), their total score, and department.

**Step 2:** In the CTE, use the `CONCAT()` function to combine the student's first name and last name with a space in between.

**Step 3:** Calculate the total score using the `SUM()` function and apply `GROUP BY` to ensure the `total_score` is aggregated at the `student_id` level.

**Step 4:** Join tables:
- Use `LEFT JOIN` to join the *Students* table with the *Studentcourse* table using `student_id`.
- Use another `LEFT JOIN` to join the *Studentcourse* table with the *Course* table using `course_id`.
  
**Note:** The step 4 is exactly the same as in Q3.

**Step 5:** Create a subquery named *subquery_1* to rank the results using the `RANK() OVER()` function.
- Set `PARTITION BY` to `department` to ensure the ranking is done within each department.
- Use `ORDER BY ... DESC` to rank by `total_score` in descending order.

**Step 6:** To enhance readability, use the `GROUP_CONCAT()` function to combine `student_id` and `student_name` into a single cell with a comma as a separator if there is a tie for the highest `total_score` within a department.

The result with `GROUP_CONCAT` function will look like:
| department | student_ids | student_name                | total_score |
|------------|-------------|-----------------------------|-------------|
| Math       | 5, 8        | Ella Fitzgerald, Hank Williams | 181         |
| Science    | 1           | Alice Cooper                | 180.5       |
| Arts       | 3           | Charlie Chaplin             | 174         |

**Step 7:** Finally, return only the student names with the highest `total_score` in each department by filtering the results where the rank is 1 in the `WHERE` clause.

```
WITH total_score_list AS (
  SELECT 
        s.student_id,
        CONCAT(s.first_name, ' ', s.surname) AS student_name,
        SUM(c.course_score) AS total_score,
        s.department
	FROM student s
  LEFT JOIN studentcourse sc ON s.student_id = sc.student_id
  LEFT JOIN course c ON sc.course_id = c.course_id
  GROUP BY student_id
    )
SELECT
      department,
      GROUP_CONCAT(DISTINCT student_id
                   ORDER BY student_id
                   SEPARATOR ', ') AS student_ids,
      GROUP_CONCAT(student_name
                   ORDER BY student_id
                   SEPARATOR ', ') AS student_name,
      total_score
FROM (
  SELECT 
        *,
        RANK() OVER(PARTITION BY department ORDER BY total_score DESC) AS rk
	FROM total_score_list
) subquery_1
WHERE rk = 1
GROUP BY department, total_score
ORDER BY total_score DESC, department;
```

## Summary
Through this project, built upon a designed database, we undertake the task of analysing and reporting key data for a language training centre. By using SQL queries, the organisation can swiftly retrieve the necessary data and perform effective data manipulation. I believe that it is a powerful tool for managing large datasets. This approach supports the ETL process and enhances efficient decision-making.

> [!NOTE]  
> The information and data in this project are entirely fictitious and are intended solely to demonstrate the benefits of SQL in data analytics.
