---
type: "practice"
difficulty: "Expert"
---

# How do you build an audit trail for who accessed which sensitive rows and when, across systems, that is tamper-evident and query-efficient at petabyte scale?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Preventing PII Leakage](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Preventing_PII_Leakage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a robust, tamper-evident, and query-efficient audit trail system for tracking who accessed which sensitive rows in multiple databases across systems, we need to ensure that our solution addresses both data integrity and performance at scale. At an architectural level, this involves creating a centralized logging mechanism, implementing strong access controls, and ensuring that every read operation is logged with sufficient metadata.

#### Triage

The question revolves around designing an audit trail system that records who accessed which sensitive rows of data across multiple databases or systems. The key aspects are:

Tamper-evident: The system must be designed to detect any unauthorized modifications.
Query-efficient: The solution should minimize performance overhead for regular operations while ensuring that auditing can be performed without significant lag.
Petabyte scale: We need to ensure the system can handle massive amounts of data.

#### Assumptions
We are dealing with multiple databases or systems, potentially in different environments (e.g., on-premises and cloud).
The sensitive rows of data must have a unique identifier that allows us to track them.
Access to these sensitive rows is logged for auditing purposes.
Performance optimizations need to be considered at scale.

#### First-Principles Solution
Centralized Logging System: Use a centralized logging system (e.g., ELK stack, Splunk) to collect and aggregate audit logs from all systems.
Unique Sensitive Row Identifiers: Ensure that every sensitive row has a unique identifier that is consistent across systems.
Access Control Mechanisms: Implement role-based access control (RBAC) or attribute-based access control (ABAC) to manage who can read, write, or modify these rows.
Log Data Integrity: Use cryptographic hashes and digital signatures to ensure the integrity of log entries.
Performance Optimization: Optimize queries for logging by using efficient indexing and caching mechanisms.
Step-by-Step Solution

Define the Audit Log Schema:

Create a schema for storing audit logs that includes fields like timestamp, user_id, resource_id, operation_type, status, etc.
SQL
CREATE TABLE audit_logs (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    timestamp TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    user_id VARCHAR(255) NOT NULL,
    resource_id VARCHAR(255) NOT NULL,
    operation_type ENUM('READ', 'WRITE') NOT NULL,
    status ENUM('SUCCESS', 'FAILURE') NOT NULL,
    additional_data JSON
);

Implement Access Control:

Use RBAC/ABAC to enforce access controls on sensitive data.
SQL
CREATE TABLE roles (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE users_roles (
    user_id INT,
    role_id INT,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (role_id) REFERENCES roles(id)
);

Ensure Data Integrity:

Use digital signatures to sign each log entry.
PYTHON
import hashlib

def generate_signature(data, secret_key):
    return hashlib.sha256((data + secret_key).encode()).hexdigest()

signature = generate_signature(log_entry['data'], 'secret_key')

Optimize Logging Queries:

Use efficient indexing and caching to minimize the overhead of logging.
SQL
-- Indexing for faster query performance
CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);

Centralized Logging Aggregation:

Use a centralized system like ELK stack to collect logs from all sources.
CODE
Filebeat -> Logstash -> Elasticsearch -> Kibana

#### Closing Summary
and Common Mistakes/Follow-Ups

The solution involves designing a robust, tamper-evident audit trail that can handle petabyte-scale data. Key aspects include centralized logging, unique identifiers for sensitive rows, strong access controls, data integrity mechanisms, and performance optimization.

#### Common mistakes
include:

Failing to implement proper indexing, leading to performance bottlenecks.
Not using digital signatures or other integrity checks, making the logs susceptible to tampering.
Underestimating the complexity of multi-system integration, which can lead to inconsistencies in log entries.

#### Follow-ups
could involve discussing specific technologies used (e.g., AWS CloudTrail, Azure Monitor), additional security measures like encryption at rest and in transit, or strategies for handling large volumes of data efficiently.

**Comparison:**

```
User	Role
user_001	admin
user_002	reader
```

**Code:**

```sql
-- Database schema creation
CREATE TABLE audit_logs (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    timestamp TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    user_id VARCHAR(255) NOT NULL,
    resource_id VARCHAR(255) NOT NULL,
    operation_type ENUM('READ', 'WRITE') NOT NULL,
    status ENUM('SUCCESS', 'FAILURE') NOT NULL,
    additional_data JSON
);

CREATE TABLE roles (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE users_roles (
    user_id INT,
    role_id INT,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (role_id) REFERENCES roles(id)
);
```

**Example input / output:**

```text
{
  "request": {
    "method": "GET",
    "url": "/api/sensitive-data/123",
    "headers": {
      "User-Agent": "curl/7.89.0",
      "Authorization": "Bearer token_12345"
    },
    "timestamp": "2023-10-05T14:48:00Z"
  },
  "response": {
    "status_code": 200,
    "data": {
      "id": 1,
      "timestamp": "2023-10-05T14:48:00Z",
      "user_id": "user_001",
      "resource_id": "sensitive_row_123",
      "operation_type": "READ",
      "status": "SUCCESS"
    },
    "timestamp": "2023-10-05T14:48:01Z"
  }
}
```

**Data:**

```json
{
  "audit_logs": [
    {
      "id": 1,
      "timestamp": "2023-10-05T14:48:00Z",
      "user_id": "user_001",
      "resource_id": "sensitive_row_123",
      "operation_type": "READ",
      "status": "SUCCESS"
    },
    {
      "id": 2,
      "timestamp": "2023-10-05T14:48:01Z",
      "user_id": "user_002",
      "resource_id": "sensitive_row_123",
      "operation_type": "READ",
      "status": "SUCCESS"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Access Request
4. Is the user authorized?
5. Fetch Sensitive Data
6. Log Audit Trail Entry
7. Deny Access and Log Denial

