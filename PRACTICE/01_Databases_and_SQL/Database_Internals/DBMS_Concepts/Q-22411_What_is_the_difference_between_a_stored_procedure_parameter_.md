---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a stored procedure parameter that is IN OUT or INOUT?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Dynamic SQL](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with database procedures in SQL, understanding how parameters are passed can be crucial for effective and efficient coding. The IN OUT (or INOUT) parameter type is a key aspect of this concept. In essence, an INOUT parameter allows you to pass both input and output values between the calling program and the stored procedure. Let's break down what IN, OUT, and INOUT parameters mean and how they are used in a database.

#### Triaging the Question

The core of this question is understanding the difference between different types of parameter passing in SQL, specifically focusing on the IN OUT or INOUT keyword. The interviewer might be probing your knowledge of:

How data flows between the calling program and stored procedure.
Your ability to distinguish between input-only (IN), output-only (OUT), and bidirectional (INOUT) parameters.

#### First Principles Solution
Input Parameters (IN): These are used when you want to pass a value from the calling program to the stored procedure, but not retrieve it back. The parameter acts as an input only.
Output Parameters (OUT): These are used when you want the stored procedure to return a value to the calling program. The value is provided by the procedure and not passed in from the outside.
INOUT Parameters: This type of parameter allows both input and output operations. You can pass an initial value from the calling program, modify it within the procedure, and then return the updated value.

#### Example

Let's consider a simple example where we want to update a student’s grade based on their name in a database:

SQL
CREATE PROCEDURE UpdateGrade(INOUT student_name VARCHAR(255), IN new_grade INT)
BEGIN
    -- Search for the student by name and get their current grade
    SET @current_grade = (SELECT grade FROM students WHERE name = student_name);

    IF @current_grade IS NULL THEN
        SET student_name = 'Student not found';
        RETURN;
    END IF;

    -- Update the grade in the database
    UPDATE students SET grade = new_grade WHERE name = student_name;
END;

In this example, student_name is an INOUT parameter because it starts as a value passed from the caller and ends up modified within the procedure. The new_grade parameter is an IN parameter because only its value can be updated by the procedure.

#### Closing Summary
and Common Mistakes
Misunderstanding INOUT: A common mistake is confusing INOUT with OUT, thinking that INOUT means both input and output but in a one-way direction.
Ignoring Context: Failing to understand which part of the procedure might need an INOUT parameter. For instance, if you are setting a value based on another table's data, it should be an IN parameter.

#### Follow-Ups
In what scenarios would you prefer using INOUT over OUT or IN?
Can you give an example where using INOUT could lead to unexpected behavior?

**Comparison:**

| ID | Name    | Grade |
|----|---------|-------|
| 1  | Alice   | 90    |

**Code:**

```sql
CREATE PROCEDURE UpdateGrade(INOUT student_name VARCHAR(255), IN new_grade INT)
BEGIN
    -- Search for the student by name and get their current grade
    SET @current_grade = (SELECT grade FROM students WHERE name = student_name);

    IF @current_grade IS NULL THEN
        SET student_name = 'Student not found';
        RETURN;
    END IF;

    -- Update the grade in the database
    UPDATE students SET grade = new_grade WHERE name = student_name;
END;
```

**Example input / output:**

```text
- **Input**: `student_name = 'Alice'`, `new_grade = 90`
- **Output**: `student_name` is updated to `'Alice'`, reflecting the new grade.
```

**Data:**

```sql
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    grade INT
);
```

**Flow:**

1. Yes
2. No
3. Yes
4. Start
5. Is student_name an INOUTparameter?
6. Initialize @current_grade
7. Check if student_name existsin students table
8. Set student_name = 'Studentnot found'; Return
9. Update grade in the database

