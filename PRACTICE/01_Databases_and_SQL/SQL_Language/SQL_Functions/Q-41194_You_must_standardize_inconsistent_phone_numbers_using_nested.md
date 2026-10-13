---
type: "practice"
difficulty: "Intermediate"
---

# You must standardize inconsistent phone numbers using nested REPLACE/REGEXP_REPLACE. What are the performance and correctness risks of doing this at query time versus at write time?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ String Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/String_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with inconsistent phone numbers, you have two main options: standardizing them either at query time using SQL functions like REPLACE or REGEXP_REPLACE, or doing it at write time during data insertion. Each approach has its own set of performance and correctness risks.

**Comparison:**

| Phone Number | Standardized Query Time       | Standardized Write Time     |
|--------------|-------------------------------|-----------------------------|
| +1-234-567-8901 | 2345678901                    | 12345678901                 |
| (234) 567-8902 | 2345678902                    | 2345678902                  |
| 234.567.8903  | 2345678903                    | 2345678903                  |
| 234 567 8904  | 2345678904                    | 2345678904                  |

**Code:**

```sql
-- Example of replacing characters in a phone number at query time
SELECT REGEXP_REPLACE(phone_number, '[^0-9]', '', 'g') AS standardized_phone FROM users;

-- Example of writing to a table with standardized phone numbers
INSERT INTO users (standardized_phone) VALUES (REPLACE(phone_number, '+', '')::bigint);
```

**Example input / output:**

```text
Input (pre-standardization):
+1-234-567-8901

Output (post-standardization at query time):
2345678901

Input (pre-standardization):
(234) 567-8902

Output (post-standardization at write time):
2345678902
```

**Data:**

```json
{
  "phone_numbers": [
    "+1-234-567-8901",
    "(234) 567-8902",
    "234.567.8903",
    "234 567 8904"
  ]
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Query Time Standardization
6. Are there many updates?
7. Standardize on-the-fly
8. Create a computed column orview
9. Write Time Standardization
10. Is data insertion frequent?
11. Insert raw data, standardize atquery time
12. Standardize during insert andstore in normalized form

