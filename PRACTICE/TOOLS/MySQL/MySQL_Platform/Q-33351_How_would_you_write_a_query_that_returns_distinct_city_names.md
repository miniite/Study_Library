---
type: "practice"
difficulty: "Basic"
---

# How would you write a query that returns distinct city names from a customers table?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Basic Query Practice](<../../../../TOOLS/MySQL/MySQL_Platform/Basic_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To return distinct city names from a customers table in MySQL, we need to use the SELECT DISTINCT statement. This is one of the fundamental concepts in SQL that helps eliminate duplicate values in a result set.

#### Triaging the Question

The question asks us to write an SQL query to retrieve unique city names from a customers table. The interviewer might be probing your understanding of basic SQL syntax and how to filter out duplicates using the DISTINCT keyword.

#### First-Principles Solution
Understand the Table Schema: Assume we have a customers table with columns such as id, name, email, and city.
Use the DISTINCT Keyword: To get unique city names, we use the SELECT DISTINCT statement.
Write the Query: The query will look like this:
SQL
SELECT DISTINCT city FROM customers;

#### Closing Summary

This query effectively retrieves all distinct (unique) city names from the customers table by leveraging the DISTINCT keyword in SQL.

#### Common Mistakes
and Follow-Ups
Mistake: Forgetting to use DISTINCT. Instead, using SELECT city FROM customers;, which would return a list of cities with possible duplicates.

#### Follow-Up
The interviewer might ask about handling ties or ensuring the order of the results. In MySQL, by default, the order of unique values in a result set may not be guaranteed. If you need to enforce an order, consider using ORDER BY:
SQL
SELECT DISTINCT city FROM customers ORDER BY city;

This ensures that the cities are returned in alphabetical order.

