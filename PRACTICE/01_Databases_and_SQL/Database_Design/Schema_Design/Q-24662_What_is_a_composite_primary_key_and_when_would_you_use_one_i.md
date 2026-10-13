---
type: "practice"
difficulty: "Intermediate"
---

# What is a composite primary key and when would you use one in a schema?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Surrogate vs Natural Keys for Lookups](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Surrogate_vs_Natural_Keys_for_Lookups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A composite primary key is a combination of two or more columns that uniquely identify each record in a table. Unlike a single-column primary key, which consists of a single column, a composite primary key ensures uniqueness across multiple fields.

#### Triaging the Question
Question Restatement: The question asks for an explanation of what a composite primary key is and scenarios where it would be beneficial.

#### Assumptions
We are dealing with relational databases.
Uniqueness across data records is crucial.
Tackling the Problem from First Principles:
Define composite keys.
Discuss their benefits and drawbacks.
Provide examples of when to use them.

#### Solution
Definition:

A composite primary key consists of multiple columns that together uniquely identify a record in a table. For example, if you have a Sales table with CustomerID, OrderDate, and ProductID as fields, a composite primary key might be formed by combining CustomerID and OrderDate.

Benefits:
Reducing Redundancy: By using multiple columns, you avoid the need for additional surrogate keys.
Improved Data Integrity: Composite keys can enforce business rules that are based on combinations of values rather than single attributes.
Normalization: They help in achieving higher levels of normalization by ensuring that the combined fields uniquely identify records.
Drawbacks:
Complexity: Management and querying complexity may increase due to the combination of multiple columns.
Performance: Indexing composite keys can sometimes affect performance, especially with large numbers of rows or complex queries.
Common Scenarios for Using Composite Primary Keys:

Transaction Records:

#### Example
A Sales table where each row represents a single sale transaction involving a specific customer on a particular date.
SQL
CREATE TABLE Sales (
  CustomerID INT,
  OrderDate DATE,
  ProductID INT,
  Quantity INT,
  PRIMARY KEY (CustomerID, OrderDate)
);

Event Logs:

#### Example
A SystemLogs table where each log entry includes a timestamp and an event type.
SQL
CREATE TABLE SystemLogs (
  Timestamp TIMESTAMP,
  EventType VARCHAR(50),
  PRIMARY KEY (Timestamp, EventType)
);

Time-Stamped Data:

#### Example
A FinancialMetrics table where each metric is recorded with a specific date and time.
SQL
CREATE TABLE FinancialMetrics (
  Date DATE,
  Time TIME,
  MetricValue DECIMAL(10, 2),
  PRIMARY KEY (Date, Time)
);

#### Closing Summary

In summary, composite primary keys are useful when you need to uniquely identify records based on multiple columns. They help in reducing redundancy and improving data integrity but come with increased complexity and potential performance impacts.

#### Common Mistakes/Follow-Ups
Incorrect Key Selection: Often, developers might mistakenly choose a combination of columns that do not provide unique identification.
Overuse of Composite Keys: While they are useful in certain scenarios, overusing them can lead to unnecessary complexity without adding value.

#### Performance Considerations
It's important to consider the performance implications when designing composite keys.

**Comparison:**

| Composite Keys Example | CustomerID | OrderDate | ProductID |
|-----------------------|------------|-----------|----------|
| Sale 1                | 101        | 2023-01-01| 201      |
| Sale 2                | 101        | 2023-01-02| 202      |
| Composite Keys Example | Timestamp         | EventType    |
|-----------------------|------------------|--------------|
| Log Entry 1           | 2023-06-01 14:30 | Login        |
| Log Entry 2           | 2023-06-01 15:00 | Logout       |

**Code:**

```sql
-- Example SQL for creating a table with a composite primary key

CREATE TABLE Sales (
  CustomerID INT,
  OrderDate DATE,
  ProductID INT,
  Quantity INT,
  PRIMARY KEY (CustomerID, OrderDate)
);

CREATE TABLE SystemLogs (
  Timestamp TIMESTAMP,
  EventType VARCHAR(50),
  PRIMARY KEY (Timestamp, EventType)
);
```

**Example input / output:**

```text
Input (Sales):
CREATE TABLE Sales (
  CustomerID INT,
  OrderDate DATE,
  ProductID INT,
  Quantity INT,
  PRIMARY KEY (CustomerID, OrderDate)
);

Output:
Table 'Sales' created with composite primary key constraint.

Input (SystemLogs):
CREATE TABLE SystemLogs (
  Timestamp TIMESTAMP,
  EventType VARCHAR(50),
  PRIMARY KEY (Timestamp, EventType)
);

Output:
Table 'SystemLogs' created with composite primary key constraint.
```

**Data:**

```json
{
  "Sales": [
    { "CustomerID": 101, "OrderDate": "2023-01-01", "ProductID": 201, "Quantity": 5 },
    { "CustomerID": 101, "OrderDate": "2023-01-02", "ProductID": 202, "Quantity": 3 }
  ],
  "SystemLogs": [
    { "Timestamp": "2023-06-01 14:30:00", "EventType": "Login" },
    { "Timestamp": "2023-06-01 15:00:00", "EventType": "Logout" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Composite Primary Key
4. Is the combination of columnsunique?
5. Use Composite PK
6. Find a unique combination oruse surrogate key
7. Create table with compositeprimary key constraint
8. Add additional fields toachieve uniqueness orintroduce a surrogate key

