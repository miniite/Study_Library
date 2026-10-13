---
type: "practice"
difficulty: "Deep"
---

# How would you support both a Type 2 history column and a fast current-value column on the same customer dimension without maintaining two tables?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SCD Type 2 vs Mini-dimensions](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/SCD_Type_2_vs_Mini-dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To support both a Type 2 history column and a fast current-value column on the same customer dimension, you need to design a hybrid schema that leverages the strengths of both Type 1 (overwrite) and Type 2 (add new rows) strategies. This approach involves creating a single table that contains all necessary information while ensuring fast access to the most recent value and maintaining an audit trail for historical changes.

#### Triaging the Question

The core of this question is about designing a data model that can support both quick retrieval of current values and accurate history tracking without duplicating tables. The interviewer is probing your understanding of normalization, performance trade-offs, and how to balance between different types of data integrity strategies.

#### First-Principles Solution

Data Model Design:

Use a single table for the customer dimension.
Include columns for current values (e.g., current_status, current_balance).
Add history tracking through additional columns (e.g., status_history, balance_history).

Current Value Column:

Implement an index or unique constraint on a combination of key columns and end_date to ensure only one row with the latest end date is marked as current.
Use a column like current_flag (e.g., true for current, false for historical).

Type 2 History Columns:

For each historical attribute, maintain separate columns that store the history of changes.

Concurrency and Performance Considerations:

Ensure proper indexing to optimize both reads (for current values) and writes (for historical records).
Implement efficient query strategies to minimize the impact on performance.

Architectural Decisions:

Decide if you need to use a separate index for fast access to the most recent value.

#### Cons
ider partitioning based on time or other relevant criteria to improve query performance.

#### Implementation
Example

Below is an example of how this schema might look in SQL:

SQL
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    current_status VARCHAR(20) CHECK (current_status IN ('active', 'inactive')),
    current_balance DECIMAL(10, 2),
    status_history VARCHAR(20) CHECK (status_history IN ('active', 'inactive')) DEFAULT NULL,
    balance_history DECIMAL(10, 2) DEFAULT NULL,
    start_date DATE NOT NULL,
    end_date DATE DEFAULT NULL,
    current_flag BOOLEAN NOT NULL CHECK (current_flag IN (true, false))
);
Explanation of the Schema
customer_id: Primary key to uniquely identify each customer.
first_name, last_name: Basic attributes stored for each customer.
current_status, status_history: Columns where current and historical status are tracked.
current_balance and balance_history: To track the latest balance and its history, respectively.
start_date, end_date: Timestamps to indicate when a change occurred. An active record has a non-null end date for previous versions.
current_flag: A boolean column that marks the current version of each customer.

#### Performance Considerations
Use an index on (customer_id, end_date DESC) to quickly fetch the most recent row for any given customer.
Ensure proper indexing and query optimization techniques are used to handle frequent reads and writes efficiently.

#### Common Mistakes
and Follow-Ups

Mistake: Overlooking Indexing:

Not creating appropriate indexes can lead to performance degradation during read operations, especially when querying the most recent records.

Mistake: Poorly Designed History Columns:

Failing to track historical changes properly can result in data loss or incorrect historical reports.

#### Follow-Up
Questions:

"How would you handle concurrent updates and ensure consistency?"
"What measures would you take to optimize storage space usage for this schema?"

By carefully balancing the need for current value access with accurate history tracking, you can design a robust and efficient data model that meets your organization's needs.

**Code:**

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    current_status VARCHAR(20) CHECK (current_status IN ('active', 'inactive')),
    current_balance DECIMAL(10, 2),
    status_history VARCHAR(20) CHECK (status_history IN ('active', 'inactive')) DEFAULT NULL,
    balance_history DECIMAL(10, 2) DEFAULT NULL,
    start_date DATE NOT NULL,
    end_date DATE DEFAULT NULL,
    current_flag BOOLEAN NOT NULL CHECK (current_flag IN (true, false))
);
```

