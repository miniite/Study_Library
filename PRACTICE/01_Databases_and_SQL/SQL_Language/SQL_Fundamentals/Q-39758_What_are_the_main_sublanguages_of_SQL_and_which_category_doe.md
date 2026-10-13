---
type: "practice"
difficulty: "Basic"
---

# What are the main sublanguages of SQL and which category does SELECT belong to?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Scalar Subqueries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

SQL (Structured Query Language) is a powerful language used for managing relational databases, consisting of several sublanguages that serve different purposes. SELECT falls under the Data Manipulation Language (DML), which focuses on retrieving and updating data in the database.

#### Triaging the Question

The question asks about the main sublanguages of SQL and specifically categorizes SELECT. This is fundamental knowledge for anyone working with databases, as understanding these categories helps in structuring queries effectively. The key idea here is to distinguish between DML (Data Manipulation Language) and Data Definition Language (DDL).

Detailed Solution:

SQL primarily comprises three major categories:

Data Definition Language (DDL): This category includes statements for defining, modifying, and managing the structure of databases.
Data Manipulation Language (DML): These are commands used to retrieve or modify data in a database.
Data Control Language (DCL): This is responsible for controlling access permissions.

SELECT belongs to DML because it's used to fetch, filter, and present the desired information from one or more tables without altering their structure.

Why SELECT is DML:
Purpose: The primary purpose of SELECT is to retrieve data from a database.
Non-Modifying Nature: Unlike statements in DDL (e.g., CREATE TABLE, ALTER TABLE) and DCL (e.g., GRANT, REVOKE), SELECT does not modify the existing data or its structure. It merely retrieves it.

#### Common Mistakes/Follow-Ups
Mistake: Thinking that SELECT is part of DDL because it can create a temporary view or table.
Correct Understanding: While CREATE VIEW and similar statements are indeed in DDL, the core functionality of SELECT remains to retrieve data.

#### Closing Summary
+ Common Follow-Ups:

Understanding the sublanguages of SQL helps in structuring your queries correctly. If you're working with a more complex database system or dealing with security controls, knowing about DCL becomes important as well. Always ensure that your queries are clear and do not inadvertently change the state of your database.

#### Conclusion

SELECT is part of the Data Manipulation Language (DML) in SQL, used for data retrieval without altering the database's structure.

