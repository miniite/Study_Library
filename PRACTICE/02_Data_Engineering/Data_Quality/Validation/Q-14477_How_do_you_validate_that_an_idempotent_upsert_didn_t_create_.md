---
type: "practice"
difficulty: "Deep"
---

# How do you validate that an idempotent upsert didn't create phantom duplicates when a job retried after a partial write?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Uniqueness Checks](<../../../../02_Data_Engineering/Data_Quality/Validation/Uniqueness_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that an idempotent upsert operation didn’t inadvertently create duplicate records during a job retry, we need to validate the data integrity and track any potential anomalies. This process involves several steps, from understanding what idempotent upserts are to implementing robust validation checks.

#### Triaging the Question

The question asks how to validate that an idempotent upsert operation didn’t create phantom duplicates when a job retried after a partial write. The key here is ensuring that the system correctly handles retries without introducing duplicate records, which could otherwise lead to data inconsistencies or performance issues.

#### Assumptions
Idempotent Upsert: This operation ensures that any number of identical upserts (insert or update) operations will result in exactly one record being modified.
Partial Write: The job did not complete successfully and was retried, leading to a potential gap in the data processing.

#### First-Principles Solution

To validate this, we need to follow these steps:

Identify Potential Duplicates:

Compare the records from the last successful write with those of the current write.

Track Retried Records:

Maintain a log or metadata table to track which records were processed during each attempt.

#### Cons
istency Checks:

Verify that no new records are inserted during retries, and existing records are not updated multiple times without proper validation.
Detailed Implementation
Step 1: Identifying Potential Duplicates

We can use a combination of primary keys and additional unique constraints to identify any potential duplicates. For example, if we have a table users with columns id, email, and name, we would compare the records based on these fields.

SQL
SELECT * FROM users WHERE id IN (SELECT id FROM partial_write)
INTERSECT
SELECT * FROM users WHERE email IN (SELECT email FROM partial_write);

This query helps us find any overlapping records that might indicate a duplicate insertion.

Step 2: Tracking Retried Records

Maintain a log table, say write_attempts, to track each job execution and the records processed:

SQL
CREATE TABLE write_attempts (
    attempt_id SERIAL PRIMARY KEY,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP,
    total_records INT NOT NULL,
    successfully_processed INT NOT NULL,
    retries INT DEFAULT 0
);

Each time a job is executed, update this table to record the number of records processed and any retries. This helps in identifying if there were any gaps or overlaps.

Step 3: Consistency Checks

Implement checks within your application logic to ensure that no new records are inserted during retries and existing records are only updated as needed. For example:

PYTHON
def upsert_user(user_data):
    # Check if user exists based on unique constraints (id, email)
    existing_user = User.objects.filter(id=user_data['id'], email=user_data['email']).first()

    if existing_user:
        # Update the existing record
        update_fields = {field: value for field, value in user_data.items() if getattr(existing_user, field) != value}
        existing_user.update(**update_fields)
    else:
        # Insert a new record
        User.objects.create(**user_data)

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary

To ensure idempotent upserts don’t create phantom duplicates during job retries, we need to implement robust validation checks, track processed records, and maintain consistency across updates. This involves comparing data from different attempts, logging each execution, and ensuring that our logic handles retries without introducing new errors.

#### Common Mistakes/Follow-Ups
Ignoring Metadata: Failing to keep a log of job executions can lead to confusion during validation.
Incomplete Data Checks: Only checking primary keys might miss duplicate entries based on unique constraints or business rules.
Race Conditions: Not handling concurrent updates properly can still introduce duplicates.

#### Follow-ups
could include discussing how to handle distributed systems, ensuring data consistency in multi-tenant environments, and optimizing performance during large-scale writes.

**Comparison:**

```
Attempt ID	Start Time	End Time	Total Records	Successfully Processed	Retries
1	2023-04-01T12:00	NULL	1000	985	0
2	2023-04-02T12:00	NULL	1000	985	1
```

**Code:**

```python
def upsert_user(user_data):
    # Check if user exists based on unique constraints (id, email)
    existing_user = User.objects.filter(id=user_data['id'], email=user_data['email']).first()

    if existing_user:
        # Update the existing record
        update_fields = {field: value for field, value in user_data.items() if getattr(existing_user, field) != value}
        existing_user.update(**update_fields)
    else:
        # Insert a new record
        User.objects.create(**user_data)
```

**Example input / output:**

```text
Input:
Partial write with records [user1, user2] in attempt_id 1.

Output (after validation):
No duplicates found between attempts. Validation passed.
```

**Data:**

```json
{
  "write_attempts": [
    {
      "attempt_id": 1,
      "start_time": "2023-04-01T12:00:00Z",
      "end_time": null,
      "total_records": 1000,
      "successfully_processed": 985,
      "retries": 0
    },
    {
      "attempt_id": 2,
      "start_time": "2023-04-02T12:00:00Z",
      "end_time": null,
      "total_records": 1000,
      "successfully_processed": 985,
      "retries": 1
    }
  ]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Start
6. Is job retry?
7. Identify duplicates and validate
8. Log attempt details
9. Consistency checks passed?
10. Validation passes, no actionneeded
11. Notify admin of issue
12. Regular job execution

