---
type: "practice"
difficulty: "Deep"
---

# What is a slowly changing dimension (SCD Type 2) and how could you implement it with MERGE in Delta?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Delta Live Tables and Serving](<../../../../TOOLS/Databricks/Databricks_Platform/Delta_Live_Tables_and_Serving.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Slowly Changing Dimension Type 2 (SCD Type 2) involves maintaining a history of changes by adding new records instead of updating existing ones. To implement SCD Type 2 using MERGE in Delta, you can leverage Delta's upsert capability to handle both insert and update operations seamlessly.

#### Triage

The question is essentially asking for an implementation strategy that leverages Databricks' Delta Lake capabilities to manage slowly changing dimensions (SCD) type 2. Specifically, the key aspects are:

Understanding SCD Type 2: Adding new records with updated timestamps instead of updating existing ones.
Using MERGE in Delta: Implementing an upsert operation to handle both insert and update operations efficiently.

#### First-Principles Solution
Step-by-Step Implementation

Schema Definition:

Define a schema for the dimension table that includes columns like id, name, version, modified_at, etc.

Staging Area:

Create a staging area to capture incoming data with versioning information.

Merge Operation:

Use MERGE in Delta Lake to handle both insert and update operations based on the existing records.

Handling New Data:

For new records, append them to the main table.
For updated records, increment the version and add a new record with the modified data.
Detailed Steps

Schema Definition

SQL
CREATE TABLE IF NOT EXISTS customers (
    id INT,
    name STRING,
    address STRING,
    version INT,
    modified_at TIMESTAMP
);

Staging Area:

Assume we have a stream of new customer data.
SQL
CREATE OR REPLACE TEMP VIEW incoming_data AS
SELECT 101 AS id, 'New Customer' AS name, 'New Address' AS address FROM VALUES (1);

Merge Operation

Define the merge operation to handle both insert and update operations.
SQL
MERGE INTO customers c
USING incoming_data i
ON c.id = i.id AND c.version = (
    SELECT MAX(version) FROM customers WHERE id = c.id
)
WHEN MATCHED THEN
    UPDATE SET name = i.name, address = i.address, modified_at = current_timestamp(), version = version + 1
WHEN NOT MATCHED THEN
    INSERT (id, name, address, version, modified_at)
    VALUES (i.id, i.name, i.address, 1, current_timestamp());

Handling New Data

For new records, simply append them to the main table.
SQL
INSERT INTO customers (id, name, address, version, modified_at)
SELECT * FROM incoming_data WHERE NOT EXISTS (
    SELECT 1 FROM customers WHERE id = incoming_data.id AND version = (
        SELECT MAX(version) FROM customers WHERE id = incoming_data.id
    )
);

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: SCD Type 2 involves maintaining a history of changes by adding new records with updated timestamps. Using MERGE in Delta Lake allows for efficient upsert operations to handle both insert and update scenarios seamlessly.

#### Common Mistakes
Forgetting to use the version column correctly during merge operations.
Not properly handling new data that does not have a matching record in the main table.
Misunderstanding the logic behind incrementing versions for updated records.
Depth + Language

The implementation details provided are detailed and match the level of an experienced engineer. The explanation includes both foundational concepts (SCD Type 2) and practical implementation using Databricks' Delta Lake.

**Comparison:**

```sql
CREATE TABLE IF NOT EXISTS customers (
    id INT,
    name STRING,
    address STRING,
    version INT,
    modified_at TIMESTAMP
);
```

**Code:**

```sql
-- Schema Definition
CREATE TABLE IF NOT EXISTS customers (
    id INT,
    name STRING,
    address STRING,
    version INT,
    modified_at TIMESTAMP
);

-- Staging Area
CREATE OR REPLACE TEMP VIEW incoming_data AS
SELECT 101 AS id, 'New Customer' AS name, 'New Address' AS address FROM VALUES (1);

-- Merge Operation
MERGE INTO customers c
USING incoming_data i
ON c.id = i.id AND c.version = (
    SELECT MAX(version) FROM customers WHERE id = c.id
)
WHEN MATCHED THEN
    UPDATE SET name = i.name, address = i.address, modified_at = current_timestamp(), version = version + 1
WHEN NOT MATCHED THEN
    INSERT (id, name, address, version, modified_at)
    VALUES (i.id, i.name, i.address, 1, current_timestamp());

-- Handling New Data
INSERT INTO customers (id, name, address, version, modified_at)
SELECT * FROM incoming_data WHERE NOT EXISTS (
    SELECT 1 FROM customers WHERE id = incoming_data.id AND version = (
        SELECT MAX(version) FROM customers WHERE id = incoming_data.id
    )
);
```

**Data:**

```sql
-- Sample Data Schema
id: INT, name: STRING, address: STRING, version: INT, modified_at: TIMESTAMP

-- Staging Area Example Data
id=101, name='New Customer', address='New Address'
```

**Flow:**

1. Yes
2. No
3. Start
4. Is there a new record?
5. Merge into main table
6. Increment version for updatedrecords
7. Insert new record in main table
8. End

