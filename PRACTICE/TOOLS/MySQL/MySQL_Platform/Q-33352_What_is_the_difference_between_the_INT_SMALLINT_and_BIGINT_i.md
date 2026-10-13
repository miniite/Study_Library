---
type: "practice"
difficulty: "Basic"
---

# What is the difference between the INT, SMALLINT, and BIGINT integer types in terms of storage?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date Arithmetic](<../../../../TOOLS/MySQL/MySQL_Platform/Date_Arithmetic.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with MySQL databases as a fresher or early-stage database developer, understanding the differences between INT, SMALLINT, and BIGINT is crucial because it helps manage memory usage efficiently without compromising on data storage requirements. In essence:

INT (4 bytes): Used for general-purpose integer values.
SMALLINT (2 bytes): Suitable for smaller integer ranges, optimizing space consumption when dealing with fewer values.
BIGINT (8 bytes): Handles larger integer values and is necessary when you need to store very high or low numbers.

To clarify what the question is asking, we're exploring how these data types differ in terms of storage capacity, range, and memory usage. The key assumptions here are that you understand the basic principles of integer types and their implications on database performance.

Let's break down each type from first principles:

INT (4 bytes):

Range: -2,147,483,648 to 2,147,483,647.
Suitable for most use cases where the values are not too large or small.

SMALLINT (2 bytes):

Range: -32,768 to 32,767.
This type is useful when you know that your integer values will always stay within this smaller range, helping reduce memory usage by half compared to INT.

BIGINT (8 bytes):

Range: -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807.
Ideal for scenarios where you need to handle extremely large numbers.

Now, let's see how we can choose the right type based on these characteristics:

If your application deals with large values (like in financial transactions or timestamps), use BIGINT.
For most general cases and medium ranges, stick with INT as it offers a good balance between range and memory usage.
When memory is a critical concern and you know the values will always be within -32k to 32k, opt for SMALLINT.

In terms of practical examples, consider storing user IDs or small numerical data like row indices. Using SMALLINT here would save space without sacrificing too much performance.

**Code:**

```sql
-- Example of declaring these integer types in a MySQL table

CREATE TABLE example (
    id INT AUTO_INCREMENT PRIMARY KEY,
    small_id SMALLINT,
    big_id BIGINT
);
```

**Example input / output:**

```sql
INSERT INTO example (small_id, big_id) VALUES (-10, -9223372036854775807);
SELECT small_id, big_id FROM example;
small_id	big_id
-10	-9223372036854775807
```

**Data:**

```text
- **Small integer range**: -32768 to 32767
- **Integer range**: -2147483648 to 2147483647
- **Big integer range**: -9223372036854775808 to 9223372036854775807
```

