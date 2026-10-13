---
type: "practice"
difficulty: "Deep"
---

# How do you handle late-arriving and slowly-corrected source data — a source fixes a typo six months later — such that the golden record heals without erasing the correct decision history?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Golden Records and Survivorship](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Golden_Records_and_Survivorship.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address late-arriving and slowly-corrected source data in Master Data Management (MDM), the key is to maintain a robust mechanism that ensures historical integrity while allowing for eventual corrections. The core idea is to store both the original and corrected versions of the golden record, along with timestamps, so that the decision history remains intact. This approach involves implementing versioning, audit trails, and change data capture (CDC) mechanisms.

First, let's restate what this question is probing:

Handling Data Integrity: Ensuring that even when late or slowly-corrected data arrives, the system can maintain a consistent view of truth.
Audit Trails: Capturing historical changes to trace back who made corrections and when.

#### Performance Considerations
Balancing between storage efficiency and query performance.

#### Triaging the Question
The question is asking how to manage and integrate late-arriving or slowly-corrected data in an MDM system without losing critical decision history.
Implicitly, it seeks a solution that balances robustness with cost-effectiveness and performance optimization.

#### First-Principles Solution
1. Data Versioning

Implement versioning for the golden record to store each state of the record over time:

SQL
CREATE TABLE Customer (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    address VARCHAR(255),
    version INT,
    last_updated TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

Each update increments the version field and sets a new last_updated timestamp. This allows tracking changes without losing historical data.

2. Audit Trails

Maintain an audit trail to record every change, including who made it:

SQL
CREATE TABLE CustomerChangeLog (
    id INT PRIMARY KEY,
    customer_id INT,
    old_name VARCHAR(255),
    new_name VARCHAR(255),
    change_type ENUM('UPDATE', 'DELETE'),
    changed_by VARCHAR(100),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

This table logs all modifications, providing a history of changes.

3. Change Data Capture (CDC)

Use CDC to capture and process changes from source systems in real-time or near-real-time:

Incremental Updates: For slowly-corrected data, ensure that the system can apply these updates incrementally.
Batch Processing: For late-arriving data, use batch processing to apply historical corrections.
4. Golden Record Aggregation

Implement a mechanism to present the most recent correct version of the record:

SQL
SELECT c.*
FROM Customer c
JOIN (
    SELECT id, MAX(version) as max_version
    FROM Customer
    GROUP BY id
) cv ON c.id = cv.id AND c.version = cv.max_version;

This query retrieves the latest version of each record based on version.

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary
By using data versioning, maintaining audit trails, implementing CDC, and ensuring proper golden record aggregation, we can effectively handle late-arriving and slowly-corrected source data while preserving decision history. This approach ensures that the system remains robust and cost-effective.

#### Common Mistakes/Follow-Ups

Data Loss: Forgetting to version or log changes can lead to losing historical context.
Performance Overhead: Storing many versions of a record can increase storage costs and query complexity.
Late Data Handling: Without proper handling, late-arriving data might be ignored, leading to inconsistent results.

Further discussion could include strategies for optimizing storage, balancing versioning retention policies, and integrating with existing systems (e.g., ETL processes).

**Comparison:**

```sql
CREATE TABLE Customer (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    address VARCHAR(255),
    version INT,
    last_updated TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE CustomerChangeLog (
    id INT PRIMARY KEY,
    customer_id INT,
    old_name VARCHAR(255),
    new_name VARCHAR(255),
    change_type ENUM('UPDATE', 'DELETE'),
    changed_by VARCHAR(100),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Code:**

```sql
-- Example SQL table creation statements
CREATE TABLE Customer (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    address VARCHAR(255),
    version INT,
    last_updated TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE CustomerChangeLog (
    id INT PRIMARY KEY,
    customer_id INT,
    old_name VARCHAR(255),
    new_name VARCHAR(255),
    change_type ENUM('UPDATE', 'DELETE'),
    changed_by VARCHAR(100),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Example input / output:**

```text
-- Example input query
SELECT c.*
FROM Customer c
JOIN (
    SELECT id, MAX(version) as max_version
    FROM Customer
    GROUP BY id
) cv ON c.id = cv.id AND c.version = cv.max_version;

-- Expected output (assuming latest version of customer with ID 1)
+----+---------+-----------+-------+-------------+---------------------+
| id | name    | address   | version | last_updated| created_at          |
+----+---------+-----------+-------+-------------+---------------------+
| 1  | John Doe| 123 Main St| 4      | 2023-10-05  | 2023-01-01 08:00:00 |
+----+---------+-----------+-------+-------------+---------------------+
```

