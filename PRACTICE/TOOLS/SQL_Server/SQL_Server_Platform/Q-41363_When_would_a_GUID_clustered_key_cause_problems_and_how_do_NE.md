---
type: "practice"
difficulty: "Intermediate"
---

# When would a GUID clustered key cause problems, and how do NEWSEQUENTIALID or a surrogate int key address them?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Clustered Indexes and Indexed Views](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/Clustered_Indexes_and_Indexed_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using SQL Server as a database developer, understanding when a GUID (Globally Unique Identifier) clustered key can lead to performance issues is crucial. A GUID clustering key often causes problems due to its lack of sequential nature and the way it affects index fragmentation and query performance. In contrast, using NEWSEQUENTIALID or an integer surrogate key can address these challenges by providing better sequential growth and improved indexing.

The core issue with a GUID clustered key is that they are generated randomly, leading to poor spatial distribution in indexes. This results in higher fragmentation and slower query performance as the table grows. Additionally, GUID keys can cause hotspots (areas of high contention) due to their random nature.

#### Triage
What’s Being Probed

The question probes your understanding of:

The characteristics and implications of using a GUID clustered key.
The performance issues related to index fragmentation and query execution.
Alternative strategies like NEWSEQUENTIALID or surrogate integer keys for addressing these problems.

#### First-Principles Solution
1. Understanding GUID Clustering

When you use a GUID as the clustered index, it has several drawbacks:

Random Distribution: GUIDs are generated randomly, leading to poor spatial distribution in the B-tree structure of the index.
Fragmentation Issues: Due to their random nature, GUID keys can lead to high fragmentation levels over time, making queries slower and more costly.
Hotspots: Randomly distributed keys can cause hotspots where multiple operations try to update the same area of the index simultaneously, leading to contention.
2. Using NEWSEQUENTIALID

NEWSEQUENTIALID() generates a GUID that is sequential based on the creation date and time. This provides a better distribution of keys over time:

Sequential Growth: The generated GUIDs are more likely to be sequential in nature, which helps in maintaining better spatial distribution.
Reduced Fragmentation: Because they follow a sequence, these keys reduce index fragmentation compared to purely random GUID values.
3. Surrogate Integer Key

Using an integer surrogate key as the primary key can be even more efficient:

Sequential Nature: Integer keys are inherently sequential and maintain good spatial distribution.
Efficient Indexing: They require less space on disk and in memory, making them faster to read and write.

#### Implementation
Example

Let’s consider a practical example where we switch from using GUID as the clustered key to using an integer surrogate key:

SQL
-- Create table with GUID clustering
CREATE TABLE Employees (
    EmployeeID UNIQUEIDENTIFIER PRIMARY KEY,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50)
);

-- Insert data and observe index fragmentation
INSERT INTO Employees (EmployeeID, FirstName, LastName)
VALUES ('8C2B3F7D-6A1E-493F-A83B-83470B6A8404', 'John', 'Doe');

-- Switch to integer surrogate key with NEWSEQUENTIALID
ALTER TABLE Employees
ALTER COLUMN EmployeeID INT IDENTITY(1, 1);

-- Insert data again and observe performance
INSERT INTO Employees (FirstName, LastName)
VALUES ('Jane', 'Smith');

#### Closing Summary

In summary, while GUID clustered keys can be tempting due to their uniqueness, they often lead to poor index fragmentation and decreased query performance. Using NEWSEQUENTIALID() or an integer surrogate key can significantly improve both the distribution of keys and overall database performance.

#### Common Mistakes/Follow-Ups

Mistake: Ignoring the importance of index maintenance and statistics updates.

#### Follow-Up
Regularly update statistics to ensure accurate query plans and consider online index operations to minimize downtime.

Mistake: Not considering the additional storage overhead of GUID keys.

#### Follow-Up
Measure the impact on storage and performance before making changes.

By understanding these nuances, you can make informed decisions about key choices in SQL Server and optimize your database for better performance.

**Code:**

```sql
CREATE TABLE Employees (
    EmployeeID UNIQUEIDENTIFIER PRIMARY KEY,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50)
);

INSERT INTO Employees (EmployeeID, FirstName, LastName)
VALUES ('8C2B3F7D-6A1E-493F-A83B-83470B6A8404', 'John', 'Doe');

ALTER TABLE Employees
ALTER COLUMN EmployeeID INT IDENTITY(1, 1);

INSERT INTO Employees (FirstName, LastName)
VALUES ('Jane', 'Smith');
```

