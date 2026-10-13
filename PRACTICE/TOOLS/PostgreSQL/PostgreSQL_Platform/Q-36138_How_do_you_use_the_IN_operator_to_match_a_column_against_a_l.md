---
type: "practice"
difficulty: "Basic"
---

# How do you use the IN operator to match a column against a list of values?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ FIRST_VALUE and LAST_VALUE](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/FIRST_VALUE_and_LAST_VALUE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To use the IN operator in PostgreSQL, you need to understand that it allows you to check if a value in a column matches any one of multiple specified values. Essentially, this is a way to filter rows where the column contains any of those specific values.

#### Triage
The question asks about using the IN operator, which means we are looking at how to compare a column's values against a list of predefined values. The IN operator checks if the value in each row’s specified column is present in the provided list of values. We need to ensure that both the column and the list of values are correctly formatted.

#### First-principles Solution
The basic syntax for using IN in PostgreSQL is as follows:

SQL
SELECT * FROM table_name WHERE column_name IN (value1, value2, ...);

Here's a step-by-step breakdown:

Identify the Column and Values: Determine which column you want to filter and what values from that list you are checking against.
Write the Query: Construct your SQL query using the IN operator inside a WHERE clause.
Enclose the List in Parentheses: The values in the list must be enclosed within parentheses.

For example, if you have a table called employees with an id column and you want to retrieve all employees whose ids are 1, 5, or 9, your query would look like this:

SQL
SELECT * FROM employees WHERE id IN (1, 5, 9);

#### Closing Summary
The IN operator in PostgreSQL is a powerful tool for filtering rows based on multiple values. By placing the column and its comparison values inside parentheses and using the IN keyword, you can efficiently query databases to retrieve specific records.

#### Common Mistakes/Follow-Ups

Forgetting Parentheses: Always ensure that your list of values is enclosed in parentheses.
Using Incorrect Data Types: Make sure the data type of the column matches the data types in the list.
Comparing with Null Values: IN will return false for any null value. If you need to include nulls, consider using IS NULL or NULLS functions.

#### Performance Considerations
For large lists, consider using a subquery or temporary table if performance is an issue.

**Code:**

```sql
SELECT * FROM employees WHERE id IN (1, 5, 9);
```

