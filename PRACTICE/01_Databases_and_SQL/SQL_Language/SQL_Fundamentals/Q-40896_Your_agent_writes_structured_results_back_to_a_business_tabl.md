---
type: "practice"
difficulty: "Deep"
---

# Your agent writes structured results back to a business table. How would you make the write idempotent so a retried LLM tool-call doesn't create duplicate records?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Unique Constraints for Dedup](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Unique_Constraints_for_Dedup.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that writing structured results from an AI agent to a business table is idempotent, we need to guarantee that retrying the write operation does not lead to duplicated entries in the database. This involves understanding the nature of the writes, leveraging unique constraints, and possibly using transactional mechanisms. The core idea is to ensure that each write attempt is treated as atomic and idempotent.

#### Triage

The question asks for a mechanism to make database writes idempotent when handling retried calls from an LLM tool. This involves understanding the nature of the data being written, ensuring uniqueness, and using appropriate database constraints and transactions. The key assumptions are that we have control over the write operations and can modify the application logic.

#### Solution
Step 1: Understand Data Structure

Assume our business table has a schema like this:

SQL
CREATE TABLE BusinessTable (
    id INT PRIMARY KEY,
    data VARCHAR(255),
    creation_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
Step 2: Add Unique Constraints

To ensure that duplicate records are not created, we can add unique constraints on the columns that identify a record. For instance, if data uniquely identifies each record, we would add:

SQL
ALTER TABLE BusinessTable ADD CONSTRAINT uc_data UNIQUE (data);
Step 3: Implement Application Logic for Idempotency

We need to ensure that our application logic checks for existence before writing. This can be done by performing a SELECT query to check if the data already exists.

PYTHON
def write_to_database(data):
    connection = get_db_connection()

    # Check if the record already exists
    cursor = connection.cursor()
    query = "SELECT 1 FROM BusinessTable WHERE data=%s"
    result = cursor.execute(query, (data,))

    if result == 0:  # Record does not exist
        insert_query = "INSERT INTO BusinessTable (data) VALUES (%s)"
        cursor.execute(insert_query, (data,))
        connection.commit()
    else:
        print(f"Record with data '{data}' already exists. Operation skipped.")
Step 4: Use Transactions for Atomicity

Using transactions ensures that the write operation is atomic and idempotent. We can use a transaction to wrap our database operations.

PYTHON
def write_to_database(data):
    connection = get_db_connection()

    with connection.cursor() as cursor:
        try:
            # Check if the record already exists
            query = "SELECT 1 FROM BusinessTable WHERE data=%s"
            result = cursor.execute(query, (data,))

            if result == 0:  # Record does not exist
                insert_query = "INSERT INTO BusinessTable (data) VALUES (%s)"
                cursor.execute(insert_query, (data,))
                connection.commit()
            else:
                print(f"Record with data '{data}' already exists. Operation skipped.")
        except Exception as e:
            connection.rollback()
            raise e

# Example usage
write_to_database("some_unique_data")

#### Closing Summary
and Common Mistakes/Follow-Ups

By adding unique constraints, checking for existence before insertion, and using transactions, we can ensure that the write operation is idempotent. Common mistakes include failing to check for record existence, not using transactions, or improperly handling exceptions during retries.

#### Follow-ups
could involve discussing how this solution scales with high concurrency or exploring alternative approaches like upserts (INSERT/UPDATE) if supported by your database system.

**Comparison:**

```
stores
BusinessTable
int
id
PK
```

**Data:**

```
data
```

**Code:**

```python
def write_to_database(data):
    connection = get_db_connection()

    # Check if the record already exists
    cursor = connection.cursor()
    query = "SELECT 1 FROM BusinessTable WHERE data=%s"
    result = cursor.execute(query, (data,))

    if result == 0:  # Record does not exist
        insert_query = "INSERT INTO BusinessTable (data) VALUES (%s)"
        cursor.execute(insert_query, (data,))
        connection.commit()
    else:
        print(f"Record with data '{data}' already exists. Operation skipped.")
```

**Example input / output:**

```

No specific input-output examples are needed for this solution, but here is a conceptual example:

SQL
Copy
INSERT INTO BusinessTable (data) VALUES ('some_unique_data');
-- Record does not exist -> Inserted successfully

SELECT * FROM BusinessTable WHERE data='some_unique_data';
-- Returns the inserted record

INSERT INTO BusinessTable (data) VALUES ('some_unique_data');
-- Record already exists -> Operation skipped.
```

**Data:**

```sql
CREATE TABLE BusinessTable (
    id INT PRIMARY KEY,
    data VARCHAR(255),
    creation_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

ALTER TABLE BusinessTable ADD CONSTRAINT uc_data UNIQUE (data);
```

**Flow:**

1. Record Exists
2. Check if Record Exists
3. Insert New Record
4. Commit Transaction
5. Log Skip Message

