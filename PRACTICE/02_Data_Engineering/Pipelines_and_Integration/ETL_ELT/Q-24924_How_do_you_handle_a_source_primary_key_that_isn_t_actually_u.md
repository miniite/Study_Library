---
type: "practice"
difficulty: "Deep"
---

# How do you handle a source primary key that isn't actually unique, and only discover it after months of loads?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Handling Source Changes and Spikes](<../../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Handling_Source_Changes_and_Spikes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with ETL or ELT processes where a supposed unique identifier (primary key) turns out to be not so, this can lead to significant issues down the line, especially if these duplicates are discovered months after initial data loads. The challenge lies in ensuring robust and reliable data integrity while maintaining efficient load times.

To handle such scenarios, we need to approach the problem from first principles:

Detecting Duplicates: Implement a mechanism to detect duplicate rows based on non-unique primary keys.
Logging and Analysis: Track these duplicates for further analysis and potential resolution.
Handling Duplicates: Decide how to handle detected duplicates—whether by updating existing records, appending new data as separate records, or flagging them for manual review.

Let's break this down step-by-step:

#### Triage
What the Question is Asking

The core of this question revolves around handling non-unique primary keys in ETL processes. The key areas are:

Detecting duplicates after initial load.
Logging and analyzing these duplicates to understand their source and impact.
Implementing strategies to manage these duplicates effectively.

#### Assumptions
You have an existing ETL pipeline with a supposed unique primary key.
This primary key is discovered to be non-unique months into the operation.
The data volume is significant, requiring efficient processing mechanisms.

#### First-Principles Solution
Step 1: Detection Mechanism

To detect duplicates, we can implement the following steps in our ETL pipeline:

Hashing: Use a hash function to generate a unique identifier for each record.
Hash Comparison: Compare the new records against an existing set of hashes stored in a database or cache.
PYTHON
def generate_hash(record):
    # Example using SHA-256
    import hashlib
    return hashlib.sha256(str(record).encode()).hexdigest()

hash_set = set()
for record in incoming_records:
    hash_value = generate_hash(record)
    if hash_value in hash_set:
        duplicate_records.append(record)
    else:
        hash_set.add(hash_value)
Step 2: Logging and Analysis

Once duplicates are detected, log them for further analysis. This can be done using a structured logging system like ELK Stack or Graylog.

JSON
{
  "timestamp": "2023-10-01T15:00:00Z",
  "duplicate_count": 5,
  "source_table": "customer_orders",
  "first_seen_at": "2023-09-01T14:00:00Z",
  "last_seen_at": "2023-10-01T14:00:00Z"
}
Step 3: Handling Duplicates

Decide on a strategy to handle the duplicates based on their nature and business requirements:

Update Existing Records: If the new data is more up-to-date, update existing records.
Append as New Records: If the original record should be preserved, append the duplicate as a separate record with a timestamp or version number.
SQL
-- Example SQL to handle duplicates
INSERT INTO customer_orders (id, order_date, amount)
SELECT id, order_date, amount FROM new_data WHERE id NOT IN (SELECT id FROM existing_records);

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that we can identify and manage non-unique primary keys effectively. However, it's crucial to:

Regularly Verify: Regularly verify the unique constraints in your data sources.
Improve Data Quality: Work with upstream teams to improve data quality and ensure unique identifiers.
Monitor Performance: Monitor performance impacts of duplicate detection mechanisms.

#### Common mistakes
include overlooking initial validation or not having a robust logging system in place. Follow-ups might involve discussing specific technologies used (e.g., Apache Beam, Kafka), real-world examples, or additional strategies for handling duplicates.

**Comparison:**

```
Duplicate Records	Timestamp	Source Table
Record 1	2023-10-01	customer_orders
Record 2	2023-10-01	customer_orders
```

**Example input / output:**

```text
{
  "timestamp": "2023-10-01T15:00:00Z",
  "duplicate_count": 5,
  "source_table": "customer_orders",
  "first_seen_at": "2023-09-01T14:00:00Z",
  "last_seen_at": "2023-10-01T14:00:00Z"
}
```

**Flow:**

1. Yes
2. No
3. Receive New Records
4. Are Duplicates Detected?
5. Log and Analyze Duplicates
6. Append as New Records orUpdate Existing
7. Handle Duplicates:Update/Append

