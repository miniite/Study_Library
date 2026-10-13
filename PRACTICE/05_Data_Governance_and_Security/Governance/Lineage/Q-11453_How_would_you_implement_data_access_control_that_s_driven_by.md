---
type: "practice"
difficulty: "Deep"
---

# How would you implement data access control that's driven by column tags rather than hardcoded grants? What's the failure mode?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Glossary Metadata and Contracts](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Glossary_Metadata_and_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of implementing data access control based on column-level tags, we need to focus on a flexible and scalable approach that can adapt to changing tag values over time without requiring manual intervention. This solution should integrate seamlessly with existing data governance frameworks while providing robust security controls.

The core idea is to use metadata-driven access policies that dynamically enforce permissions based on the tags associated with each column in the database. This approach ensures that data access is granular and adaptable, aligning closely with best practices in modern data engineering.

First, let's break down what the question is really asking:

The interviewer is probing our understanding of dynamic security controls.
We need to consider how metadata can be leveraged for policy enforcement.

#### Assumptions
include a relational database management system (RDBMS) and support for metadata-driven policies.

Now, we will reason from first principles to build this solution step-by-step:

Metadata Model: Define the metadata schema that includes columns, tags, and associated access controls.
Policy Engine: Develop an engine that can read these tags and enforce corresponding access rules.
Integration with RDBMS: Ensure seamless integration with the existing database system to dynamically apply policies.
Auditing and Logging: Implement mechanisms for auditing and logging access decisions to ensure compliance.
Step-by-Step Solution
1. Metadata Model

We need a robust metadata model that captures column-level tags and associated access policies. For example:

SQL
CREATE TABLE IF NOT EXISTS column_metadata (
    column_name VARCHAR(255) NOT NULL,
    tag_name VARCHAR(255) NOT NULL,
    policy_rule TEXT NOT NULL,
    PRIMARY KEY (column_name, tag_name)
);

This table stores the tags and corresponding policies for each column.

2. Policy Engine

The policy engine will read the metadata and enforce access rules dynamically. A sample implementation could be:

PYTHON
import sqlite3

def check_access(column_name, user_id):
    conn = sqlite3.connect('metadata.db')
    cursor = conn.cursor()

    # Fetch the relevant policy rule for the column
    query = "SELECT policy_rule FROM column_metadata WHERE column_name=?"
    cursor.execute(query, (column_name,))
    policy_rule = cursor.fetchone()[0]

    if not policy_rule:
        raise ValueError(f"No policy found for column: {column_name}")

    # Parse and apply the policy rule based on user_id
    # Example: policy_rule could be a JSON string containing conditions like {"user_role": "admin"}
    import json
    rule = json.loads(policy_rule)

    if not verify_user_conditions(user_id, rule):
        raise PermissionError("Access denied")

    conn.close()
3. Integration with RDBMS

To integrate this policy engine with the database, we can intercept SQL queries at the database driver level or use a stored procedure approach:

SQL
CREATE PROCEDURE get_data_with_policy(column_name VARCHAR(255), user_id INT) AS
BEGIN
    -- Fetch data while applying metadata-driven policies
END;

This stored procedure would call our Python policy engine function and only return data if the access is granted.

4. Auditing and Logging

Implement logging mechanisms to track access decisions:

PYTHON
def log_access(column_name, user_id, result):
    with open('access_logs.txt', 'a') as f:
        f.write(f"{column_name},{user_id},{result}\n")

This logs each access decision for audit purposes.

Failure Modes
Metadata Inconsistency: If the metadata is outdated or inconsistent, it can lead to incorrect policy enforcement.
Performance Impact: Dynamic query interception and execution of policies could introduce performance overhead.
Policy Rule Errors: Incorrectly formulated policy rules can result in either overly permissive or restrictive access.

#### Closing Summary

By implementing a metadata-driven approach for dynamic data access control, we ensure that security policies are granular and adaptable. This solution leverages existing database capabilities while adding a layer of flexibility through metadata management. However, it’s crucial to carefully manage the failure modes to maintain system integrity and performance.

**Comparison:**

    | column_name | tag_name | policy_rule |
    | customer_id | sensitive | {"user_role": "admin"} |
    | account_balance | sensitive | {"user_role": "admin"} |
    | transaction_id | public | {"user_role": "reader"} |

**Code:**

```python
import sqlite3

def check_access(column_name, user_id):
    conn = sqlite3.connect('metadata.db')
    cursor = conn.cursor()

    # Fetch the relevant policy rule for the column
    query = "SELECT policy_rule FROM column_metadata WHERE column_name=?"
    cursor.execute(query, (column_name,))
    policy_rule = cursor.fetchone()[0]

    if not policy_rule:
        raise ValueError(f"No policy found for column: {column_name}")

    # Parse and apply the policy rule based on user_id
    import json
    rule = json.loads(policy_rule)

    if not verify_user_conditions(user_id, rule):
        raise PermissionError("Access denied")

    conn.close()

def verify_user_conditions(user_id, rule):
    # Example implementation of verifying conditions
    return True  # Placeholder logic

def log_access(column_name, user_id, result):
    with open('access_logs.txt', 'a') as f:
        f.write(f"{column_name},{user_id},{result}\n")
```

**Example input / output:**

```

Dry-run example:

PYTHON
Copy
# Sample data for testing
metadata = [
    ("customer_id", "sensitive", '{"user_role": "admin"}'),
    ("account_balance", "sensitive", '{"user_role": "admin"}'),
    ("transaction_id", "public", '{"user_role": "reader"}')
]

for column, tag, rule in metadata:
    check_access(column, 100)
    log_access(column, 100, 'granted')

# Expected output: Access granted for all columns
```

**Data:**

```sql
CREATE TABLE IF NOT EXISTS column_metadata (
    column_name VARCHAR(255) NOT NULL,
    tag_name VARCHAR(255) NOT NULL,
    policy_rule TEXT NOT NULL,
    PRIMARY KEY (column_name, tag_name)
);
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. User initiates query
6. Is column tagged?
7. Check tag against policy
8. Does policy permit access?
9. Grant access
10. Deny access
11. Directly execute query
12. Audit log
13. Audit log

