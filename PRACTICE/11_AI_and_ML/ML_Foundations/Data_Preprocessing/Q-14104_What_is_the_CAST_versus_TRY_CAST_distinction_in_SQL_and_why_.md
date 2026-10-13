---
type: "practice"
difficulty: "Basic"
---

# What is the CAST versus TRY_CAST distinction in SQL and why is TRY_CAST safer when cleaning dirty numeric strings?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Cleaning Text and Numeric Fields](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Cleaning_Text_and_Numeric_Fields.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, CAST and TRY_CAST are both used to convert a string into a numeric type, such as an integer or float. However, they differ in their behavior when faced with input that cannot be converted successfully. The key distinction is that TRY_CAST returns NULL for invalid inputs, whereas CAST throws an error. This makes TRY_CAST safer and more robust for cleaning dirty data.

#### Triaging the Question

The question revolves around understanding the nuances between CAST and TRY_CAST in SQL, specifically within the context of converting string data to numeric types during data preprocessing and cleaning. The interviewer is likely probing your ability to handle error-prone data and your knowledge of SQL functions. We will assume that we are working with a dataset containing strings that may or may not be valid numbers.

#### First-Principles Solution

Understanding CAST:

CAST is a strict conversion function.
It converts the input string to the specified data type.
If the string cannot be converted, it throws an error.

Understanding TRY_CAST:

TRY_CAST also attempts to convert the input string to the specified data type.
However, if the conversion fails, instead of throwing an error, it returns NULL.

Why TRY_CAST is Safer:

In real-world datasets, strings often contain invalid characters that cannot be converted directly to numbers (e.g., commas, letters, or special symbols).
Using CAST, any such string would cause the query to fail.
With TRY_CAST, these problematic strings will simply return NULL, allowing the rest of your data processing pipeline to continue without interruption.

#### Example
Use Case:

SQL
SELECT TRY_CAST(column_name AS INT) AS cleaned_column FROM table_name;

This statement tries to convert each value in column_name to an integer and returns NULL for any values that cannot be converted, ensuring the query completes successfully even with dirty data.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, using TRY_CAST is safer when cleaning numeric strings because it handles invalid inputs by returning NULL instead of causing errors. This makes your data preprocessing and cleaning processes more robust and less prone to failure due to unexpected or incorrectly formatted data.

#### Common Mistakes

Using CAST exclusively without considering the risk of encountering invalid input.
Overlooking the importance of handling potential errors in large datasets, which could lead to query failures and incomplete processing.

#### Follow-Ups

How would you handle NULL values returned by TRY_CAST?
Can TRY_CAST be used for other types of conversions besides numeric data?

**Code:**

```sql
-- Example usage of TRY_CAST
SELECT column_name, TRY_CAST(column_name AS INT) as cleaned_column FROM sample_table;
```

**Example input / output:**

```text
-- Sample input and output
Input:
| column_name  |
|--------------|
| 123          |
| test         |
| 456.78       |

Output:
| column_name | cleaned_column |
|-------------|----------------|
| 123         |               123 |
| test        | NULL           |
| 456.78      | NULL           |
```

**Flow:**

1. CastFlow
2. Yes
3. No
4. Success
5. Failure
6. Input String
7. Is the string a valid number?
8. Convert to INT
9. Return NULL
10. Input String
11. Try Convert to INT UsingTRY_CAST
12. Value converted to INT
13. NULL returned
14. Proceed with next step
15. Handle NULL values ifnecessary

