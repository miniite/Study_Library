---
type: "practice"
difficulty: "Intermediate"
---

# How do you represent a date dimension key: as a real date, an integer like 20260706, or a surrogate?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Date Dimensions](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Date_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When representing a date dimension key in data modeling, the choice between using a real date (YYYY-MM-DD), an integer (like YMD format 20260706), or a surrogate (an auto-incremented integer) depends on your specific requirements and constraints. For an intermediate-level database developer, let's break down each option and discuss the trade-offs.

Firstly, consider using a real date like YYYY-MM-DD. This approach is straightforward and aligns with how dates are typically stored in databases. However, it can lead to large storage overhead and makes querying more complex since you need to convert between text and date types frequently. For instance, if you have a table with millions of records, storing each date as 10 bytes (assuming VARCHAR format) will consume significant space.

Secondly, using an integer like YMD (e.g., 20260706) can save storage and simplify queries. This method allows for efficient sorting and filtering operations because integers are natively supported by most database engines. However, it may not be as intuitive to understand the date at a glance compared to human-readable formats.

Lastly, using a surrogate (an auto-incremented integer) is often chosen in star or snowflake schemas due to its performance benefits. Surrogates can improve query performance and simplify maintenance tasks such as data archiving or partitioning. However, they lack the inherent meaning of real dates, making them less useful for reporting and analytics without additional metadata.

Given these considerations, a good practice is to use an integer surrogate (YMD format) for date dimension keys in a star schema, especially when performance is critical. This approach balances storage efficiency with query speed. For example, you can use the following SQL snippet to generate such a key:

SQL
SELECT DATE_FORMAT(CAST(TO_DATE('2026-07-06', 'YYYY-MM-DD') AS BIGINT), 'Ymd')

This function converts the date into an integer in YMD format.

In summary, while real dates are readable and maintainable, integers offer better performance. Surrogates are ideal for complex data models where efficient querying is crucial.

**Code:**

```sql
SELECT DATE_FORMAT(CAST(TO_DATE('2026-07-06', 'YYYY-MM-DD') AS BIGINT), 'Ymd')
```

**Example input / output:**

```

Input:

SQL
Copy
SELECT DATE_FORMAT(CAST(TO_DATE('2026-07-06', 'YYYY-MM-DD') AS BIGINT), 'Ymd')

Output:

CODE
Copy
20260706
```

