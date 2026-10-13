---
type: "practice"
difficulty: "Deep"
---

# How would you write SQL to implement a Type 2 slowly changing dimension by closing the old row and inserting a new one with effective dates?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Joining Type 2 Dimensions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Joining_Type_2_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a Type 2 slowly changing dimension (SCD) using SQL, we need to maintain historical records of changes in our data by updating existing rows or inserting new ones while keeping track of when each change happened. The key challenge is ensuring that the current state of the data is always available without duplicating unnecessary information.

#### Triaging the Question

The question is essentially asking for a SQL solution that updates an existing row to mark it as inactive (closed) and inserts a new row with updated values, along with appropriate effective dates. We need to consider assumptions like whether the data schema already includes fields for tracking historical changes or if we need to add them.

Step-by-Step Solution from First Principles
Identify Existing Schema: Assume the existing table structure has columns such as id, name, status, and timestamps (start_date and end_date) that track the version of each record.
Mark Old Row As Inactive: Update the old row by setting its end date to the current timestamp, marking it as inactive.
Insert New Row with Updated Values: Insert a new row with the updated values along with the start date being the current timestamp and the end date set to indicate that this is an open-ended record (e.g., '9999-12-31').
Full SQL Implementation

Let's assume we have a table named customer_dim with columns: id, name, status, start_date, and end_date.

SQL
-- Mark the old row as inactive by setting its end date to the current timestamp
UPDATE customer_dim
SET end_date = CURRENT_TIMESTAMP
WHERE id = '123' AND status = 'Active';

-- Insert a new row with updated values, starting from the current timestamp
INSERT INTO customer_dim (id, name, status, start_date, end_date)
VALUES ('123', 'John Doe', 'Updated Status', CURRENT_TIMESTAMP, '9999-12-31');

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, to implement a Type 2 SCD, we first deactivate the old version of the record by setting its end_date to the current timestamp. Then, we insert a new row with updated values and an open-ended end_date. This approach ensures that historical versions are preserved while maintaining a single active record.

#### Common mistakes
include:

Failing to update the end_date properly before inserting a new row.
Not setting the end_date to the correct value (e.g., using '9999-12-31' for open-ended records).
Missing or incorrect use of timestamps in SQL commands.
Additional Considerations

For more complex scenarios, you might need additional logic such as handling concurrent updates or ensuring data consistency across multiple tables. It's also important to index the start_date and end_date columns to optimize performance during query execution.

**Comparison:**

| Column Name | Data Type   |
|-------------|------------|
| id          | VARCHAR(50) |
| name        | VARCHAR(100) |
| status      | VARCHAR(50) |
| start_date  | TIMESTAMP  |
| end_date    | TIMESTAMP  |

**Code:**

```sql
-- Mark the old row as inactive by setting its end date to the current timestamp
UPDATE customer_dim
SET end_date = CURRENT_TIMESTAMP
WHERE id = '123' AND status = 'Active';

-- Insert a new row with updated values, starting from the current timestamp
INSERT INTO customer_dim (id, name, status, start_date, end_date)
VALUES ('123', 'John Doe', 'Updated Status', CURRENT_TIMESTAMP, '9999-12-31');
```

**Example input / output:**

```text
Input:
- Update existing row with ID '123' and set its end date to CURRENT_TIMESTAMP.
- Insert a new row for the same ID with updated values, setting start date as CURRENT_TIMESTAMP and end date as '9999-12-31'.

Output:
- Existing row: id='123', name='John Doe', status='Active', start_date=YYYY-MM-DD HH:MM:SS, end_date=YYYY-MM-DD HH:MM:SS (current timestamp).
- New row: id='123', name='John Doe', status='Updated Status', start_date=YYYY-MM-DD HH:MM:SS, end_date='9999-12-31'.
```

**Data:**

```sql
-- Example table schema
CREATE TABLE customer_dim (
    id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(100),
    status VARCHAR(50),
    start_date TIMESTAMP,
    end_date TIMESTAMP
);
```

**Flow:**

1. Yes
2. No
3. Mark Old Row Inactive
4. Update Query?
5. Set end_date =CURRENT_TIMESTAMP
6. Insert New Row with UpdatedValues
7. Execute Update Query
8. Set start_date =CURRENT_TIMESTAMP,end_date = '9999-12-31'
9. Execute Insert Query
10. Check Data Integrity andConsistency

