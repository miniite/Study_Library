---
type: "practice"
difficulty: "Basic"
---

# What does a NULL value mean conceptually, and why is it not the same as zero or an empty string?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ NULL Handling](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/NULL_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with SQL databases, NULL can be a bit tricky to understand because it's not quite like other data types such as integers, strings, or booleans. Conceptually, NULL represents the absence of a value, meaning that there is no information available or stored for this field. In practice, it often indicates that the value is unknown, missing, or irrelevant.

The question asks why NULL is not the same as zero or an empty string. To understand this, we need to break down what these terms mean in SQL and how they are handled differently by the database engine.

Firstly, let's consider zero:

In a numeric context, zero (0) means that there is a specific value present.
It indicates that you have something of quantity or amount.
For example, if you're storing the number of items in stock for an item, 0 would mean "none" but still represents a valid count.

Now, let's look at an empty string:

In a text context, an empty string ('' or "") is explicitly defined as having no characters.
It means that there is some kind of data present, just not any content.
For instance, if you have a column for additional notes and the field has nothing entered, it's still a valid piece of information (albeit an empty one).

In contrast, NULL:

Represents a missing or unknown value.
It indicates that there is no data stored at all for this particular attribute.
For example, if you have a customer table with columns for first_name, last_name, and address, and a record has no address information entered, then the address field should be marked as NULL.

To illustrate why NULL is different from zero or an empty string, consider a practical scenario:

If your database stores user data, including their age. A user who hasn't been asked for their age might have their age stored as NULL. This means the value is missing, not that they are 0 years old.
Similarly, if you have a table of transactions where some transactions may not have any associated notes but have other information like date and amount, an empty string in this context would indicate no specific note was added, whereas NULL would mean there were no notes at all.

These differences matter because SQL treats them differently in various operations:

Arithmetic: NULL + 10 is still NULL, not 10.
String concatenation: '' || 'world' results in 'world', but NULL || 'world' also remains NULL.
Comparisons: WHERE column = NULL does not return any rows, because NULL cannot be directly compared with other values.

It's important to use the right value depending on what you want to represent:

Use NULL when there is no information available.
Use zero or an empty string when a specific value (even if it’s "nothing") should be represented.

**Code:**

```sql
-- Example usage of NULL, 0, and ''
SELECT
    age,
    notes
FROM
    users;

-- Inserting values into the users table
INSERT INTO users (age, notes) VALUES (18, 'Hello');
INSERT INTO users (age) VALUES (NULL);
INSERT INTO users (notes) VALUES ('');
```

**Example input / output:**

```text
-- Example query to demonstrate usage of NULL in comparison
SELECT
    age,
    notes
FROM
    users
WHERE
    age = NULL;  -- This will not return any rows

SELECT
    age,
    notes
FROM
    users
WHERE
    age IS NULL;  -- This correctly returns the rows where age is NULL
```

**Data:**

```json
{
  "users": [
    { "id": 1, "name": "Alice", "age": null, "notes": "" },
    { "id": 2, "name": "Bob", "age": 0, "notes": "First transaction" }
  ]
}
```

