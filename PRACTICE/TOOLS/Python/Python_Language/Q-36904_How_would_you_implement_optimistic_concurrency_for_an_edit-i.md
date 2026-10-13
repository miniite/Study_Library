---
type: "practice"
difficulty: "Deep"
---

# How would you implement optimistic concurrency for an edit-in-place UI backed by a Python API, so two users editing the same record get a clear conflict instead of a silent overwrite?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Global Quotas Across Requests](<../../../../TOOLS/Python/Python_Language/Global_Quotas_Across_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Optimistic concurrency is crucial in ensuring data consistency when multiple users edit the same resource concurrently. For an edit-in-place UI backed by a Python API, we need to implement a version number or timestamp-based mechanism to detect conflicts before applying changes. Here's how you can achieve this:

TL;DR

We will use a version number stored with each record and increment it on every update. When a user submits a change, the backend checks if the submitted version matches the current one in the database. If they don't match, it indicates that another user has edited the record since the last fetch.

#### Triaging the Question

The core of this question is understanding how to handle concurrent updates without causing silent overwrites. The interviewer might be probing your knowledge on:

Versioning and optimistic locking.
Database interactions in Python.
API design for concurrency control.

#### First-Principles Solution
Step 1: Define the Data Model

Each record will have an additional version field, which increments with every update operation.

Yes
No
Yes
User Fetches Record
Database Returns Record
Record Version Match?
User Edits Record
User Submits Changes
Backend Checks Version Match
Version Matches?
Conflict Detected, Show UserError
Apply User's Changes toDatabase
Step 2: Implement the Backend Logic

Here’s a simplified version of what the backend might look like in Python:

PYTHON
def update_record(record_id, new_data):
    # Fetch the record with its current version
    record = get_record_by_id(record_id)

    if not record:
        raise RecordNotFound("Record does not exist")

    # Check if the submitted version matches the current one
    if record["version"] != request.form.get('version'):
        return {"error": "Concurrency error, please refresh and retry"}, 409

    # Increment the version number for this update
    record["version"] += 1
    new_data['version'] = record["version"]

    # Update the database with the new data
    save_record(record_id, new_data)

    return {"message": "Record updated successfully"}, 200

def get_record_by_id(id):
    # Dummy implementation to fetch a record from the database
    records = [
        {'id': 1, 'name': 'John Doe', 'version': 3},
        {'id': 2, 'name': 'Jane Smith', 'version': 5}
    ]
    return next((r for r in records if r['id'] == id), None)

def save_record(id, data):
    # Dummy implementation to save a record back to the database
    pass

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary

By using optimistic concurrency control with version numbers, we ensure that multiple users can edit records safely without overwriting each other's changes. The backend checks the submitted version against the current one before applying updates.

#### Common Mistakes
and Follow-Ups
Overlooking Conflict Resolution: Ensure you handle conflicts gracefully by showing the user a clear error message.
Improper Version Incrementing: Make sure to increment the version number correctly with every update operation.
API Design: Consider how your API communicates errors effectively, especially HTTP status codes and error messages.
Database Performance: For large-scale applications, consider database transactions or distributed locking mechanisms to handle concurrency more robustly.

Would you like to explore additional scenarios or discuss trade-offs between optimistic and pessimistic concurrency?

**Comparison:**

```
Record ID	Name	Version
1	John Doe	3
2	Jane Smith	5
```

**Code:**

```python
def update_record(record_id, new_data):
    # Fetch the record with its current version
    record = get_record_by_id(record_id)

    if not record:
        raise RecordNotFound("Record does not exist")

    # Check if the submitted version matches the current one
    if record["version"] != request.form.get('version'):
        return {"error": "Concurrency error, please refresh and retry"}, 409

    # Increment the version number for this update
    record["version"] += 1
    new_data['version'] = record["version"]

    # Update the database with the new data
    save_record(record_id, new_data)

    return {"message": "Record updated successfully"}, 200

def get_record_by_id(id):
    # Dummy implementation to fetch a record from the database
    records = [
        {'id': 1, 'name': 'John Doe', 'version': 3},
        {'id': 2, 'name': 'Jane Smith', 'version': 5}
    ]
    return next((r for r in records if r['id'] == id), None)

def save_record(id, data):
    # Dummy implementation to save a record back to the database
    pass
```

**Example input / output:**

```text
# Example input and output

record_id = 1
new_data = {'name': 'John Updated'}
response = update_record(record_id, new_data)
print(response)  # Output: {"message": "Record updated successfully"}

# Simulate a conflict scenario
conflict_response = update_record(record_id, {'name': 'John Updated', 'version': 4})
print(conflict_response)  # Output: {"error": "Concurrency error, please refresh and retry"}, 409
```

**Data:**

```json
{
    "records": [
        {"id": 1, "name": "John Doe", "version": 3},
        {"id": 2, "name": "Jane Smith", "version": 5}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. User Fetches Record
5. Database Returns Record
6. Record Version Match?
7. User Edits Record
8. User Submits Changes
9. Backend Checks Version Match
10. Version Matches?
11. Conflict Detected, Show UserError
12. Apply User's Changes toDatabase

