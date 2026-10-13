---
type: "practice"
difficulty: "Intermediate"
---

# How do you convert a string to a date in Oracle using TO_DATE and format masks?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Oracle Data Types](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_Data_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To convert a string to a date in Oracle, we use the TO_DATE function along with a format mask that specifies how the input string is formatted. The key idea here is to ensure that the format mask correctly matches the structure of the input string. Here’s a step-by-step explanation and a practical example.

**Code:**

```sql
-- Example string and conversion using TO_DATE
SELECT TO_DATE('2023-10-05', 'YYYY-MM-DD') AS converted_date FROM dual;

This query converts the string '2023-10-05' to a date with the format mask 'YYYY-MM-DD'.
```

**Example input / output:**

```text
| Input String | Format Mask   | Output Date       |
|--------------|---------------|-------------------|
| 2023-10-05   | YYYY-MM-DD    | 05-OCT-2023       |

This table illustrates the conversion of a string to a date using the `TO_DATE` function and format mask.
```

