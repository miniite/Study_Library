---
type: "practice"
difficulty: "Intermediate"
---

# What does the TO_DATE function do and how do you parse a string into a date with it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Date_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To_DATE is an essential SQL function used to convert a string representation of a date into a DATE data type, allowing more complex date calculations in queries. The core idea is that you provide a string and specify the format mask to correctly interpret the string as a date.

The question essentially asks two things:

What does TO_DATE do?
How to parse a string into a date using TO_DATE?

Let's break this down step-by-step, starting from first principles.

**Comparison:**

| Input String | Format Mask         | Parsed Date  |
|--------------|---------------------|--------------|
| '2023-10-05'  | 'YYYY-MM-DD'        | 2023-10-05   |
| '10/05/2023'  | 'MM/DD/YYYY'        | 2023-10-05   |
| '05-10-2023'  | 'DD-MM-YYYY'        | 2023-10-05   |

**Code:**

```

-- Example usage of TO_DATE function SELECT TO_DATE('2023-10-05', 'YYYY-MM-DD') AS parsed_date FROM dual;

CODE
Copy
This query converts the string `'2023-10-05'` into a DATE data type, using the format mask `YYYY-MM-DD`.

### Triage
The interviewer is probing your understanding of:
- **Date and String Conversion:** How to transform a date represented as a string into a proper SQL DATE data type.
- **Format Masks:** The syntax and common usage patterns of TO\_DATE.

### Assumptions
- You understand basic SQL date handling.
- You are familiar with format masks for dates in your database environment (e.g., Oracle, PostgreSQL).

### Solution from First Principles
The `TO_DATE` function takes two arguments:
1. **String Value:** The string that represents the date.
2. **Format Mask:** A string that defines how the input value is formatted.

Here's a step-by-step explanation of the process:

1. **Input String**: This is your date as a string, like `'2023-10-05'`.
2. **Format Mask**: This specifies the exact format of the date in the string.
   - `YYYY` for 4-digit year
   - `MM` for 2-digit month
   - `DD` for 2-digit day

### Example Walkthrough
Let's say we have a string `'10/5/2023'`, and you want to convert it into a date. Here’s the correct format mask:

```sql
SELECT TO_DATE('10/5/2023', 'MM/DD/YYYY') AS parsed_date FROM dual;

This converts the input string into a proper DATE data type.

Common Mistakes and Follow-Ups
Incorrect Format Mask: Using a format mask that doesn't match your string can lead to errors or incorrect date values.
Ambiguous Dates: If multiple formats could apply (e.g., '10/5' could be 2010-05-01 or 2023-10-05), the function might produce unexpected results.

Follow-Up Questions:

How would you handle ambiguous dates?
What are some common format masks used in your database environment?
Closing Summary

In conclusion, TO_DATE is a powerful tool for converting date strings into SQL DATE data types. It requires specifying a format mask that matches the input string to ensure correct parsing. Understanding and using this function correctly can significantly enhance query flexibility and accuracy.

SQL
Copy
-- Example of handling an ambiguous date
SELECT TO_DATE('10/5', 'DD/MM') AS parsed_date FROM dual;  -- Ambiguous, could be either DD/MM or MM/DD

SELECT TO_DATE('2023-10-05', 'YYYY-MM-DD') AS parsed_date FROM dual;  -- Correct format
```

**Example input / output:**

| Input String | Format Mask         | Parsed Date  |
|--------------|---------------------|--------------|
| '2023-10-05'  | 'YYYY-MM-DD'        | 2023-10-05   |
| '10/05/2023'  | 'MM/DD/YYYY'        | 2023-10-05   |
| '05-10-2023'  | 'DD-MM-YYYY'        | 2023-10-05   |

**Data:**

```sql
-- Sample input strings and their correct parsing
'2023-10-05'
'10/05/2023'
'05-10-2023'
```

**Flow:**

1. Input String
2. Specify Format Mask
3. Run TO_DATE Function
4. Parsed Date Output
5. Verify Correctness withExamples

