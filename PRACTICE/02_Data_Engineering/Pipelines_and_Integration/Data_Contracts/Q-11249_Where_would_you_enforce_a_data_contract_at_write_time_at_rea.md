---
type: "practice"
difficulty: "Deep"
---

# Where would you enforce a data contract: at write time, at read time, or in CI? Argue the trade-offs.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Contract Practice](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Contracts/Data_Contract_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce a data contract effectively, we need to consider where it makes the most sense within our software and deployment pipeline—whether that’s at write time, read time, or during CI (Continuous Integration). Enforcing contracts can ensure data integrity and consistency, but each approach comes with trade-offs. In this response, I’ll explore these points from first principles.

Firstly, let's clarify what we mean by a "data contract." A data contract specifies the structure, format, and semantics of data. For instance, it might define that a user record must have certain fields like username, email, etc., or that all dates are in ISO 8601 format.

At write time, enforcing contracts typically happens during data ingestion or when records are created. This can be done using validation frameworks and rules defined by the application or database schema. For example, a REST API might validate incoming JSON payloads before storing them in a database. The primary benefits of this approach include early error detection and ensuring that only valid data reaches storage, thereby reducing downstream issues.

At read time, enforcement is about validating data as it’s being accessed or transformed. This can happen when data is queried from a database or processed by a pipeline. For example, a BI tool might validate the structure of a dataset before performing analysis. Enforcing contracts at this stage ensures that queries and transformations are based on valid data, but it may come with performance overhead since validation needs to be done for every read operation.

In CI, enforcement is typically part of automated testing and validation processes. This involves running checks on code changes to ensure they don’t introduce any invalid data structures or behaviors. CI enforcement can catch issues early in the development process, making it easier to resolve them before deploying new versions of software. However, overly strict contract enforcement in CI might slow down the build times, so it’s important to balance comprehensiveness with performance.

The choice between these approaches depends on several factors:

Performance: Writing-time validation can be faster since it occurs once per write, whereas read-time validation is performed for every query.
Error Context: Writing-time validation provides immediate context about the issue (e.g., in the application logs), while read-time validation might not give as clear information unless detailed logging is implemented.
Scope of Impact: CI enforcement impacts all changes before deployment, which can be more comprehensive but also more disruptive.

To illustrate these points, let’s consider a real-world example. Suppose we have a microservice that ingests user data and stores it in a database. We could enforce contracts at the API level to validate incoming JSON payloads (write-time), use stored procedures or triggers to validate data as it is inserted into the database (write-time), or perform additional validation during read operations to ensure data integrity for reporting purposes (read-time). Alternatively, we could include contract checks in our CI pipeline to catch any issues introduced by new code changes (CI).

In summary, while write-time and CI enforcement are more proactive in preventing invalid data, they may introduce overhead. Read-time enforcement can be slower but ensures that all access to the data is valid.

**Comparison:**

```
Field	Validation Method
Username	Write-time, Read-time, CI
Email	Write-time, Read-time, CI
Created At	Write-time, Read-time
```

**Code:**

```python
# Example of writing-time validation using a Python function
def validate_user_data(data):
    required_fields = {'username', 'email'}
    if not required_fields.issubset(data.keys()):
        raise ValueError("Missing required fields")

    # Additional checks
    try:
        datetime.strptime(data['created_at'], '%Y-%m-%d')
    except ValueError:
        raise ValueError("Invalid date format")

# Example of read-time validation using SQL query with a check constraint
CREATE TABLE users (
    username VARCHAR(255),
    email VARCHAR(255),
    created_at TIMESTAMP,
    CHECK (username IS NOT NULL AND email IS NOT NULL)
);
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "user": {
    "username": "john_doe",
    "email": null,
    "created_at": "2023-10-01"
  }
}

Output:

CODE
Copy
ValueError: Missing required fields
```

**Data:**

```json
{
  "user": {
    "username": "john_doe",
    "email": "johndoe@example.com",
    "created_at": "2023-10-01"
  }
}
```

**Flow:**

1. Write Time
2. Validate Data
3. Store Valid Data
4. Reject Invalid Data
5. Read Time
6. Query Data
7. Validate Read Only
8. Use Valid Data for Processing
9. CI
10. Run Validation Checks
11. Promote to Production
12. Reject Invalid Changes

