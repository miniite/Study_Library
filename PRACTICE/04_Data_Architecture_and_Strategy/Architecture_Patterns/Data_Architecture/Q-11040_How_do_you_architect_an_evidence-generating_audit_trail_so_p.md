---
type: "practice"
difficulty: "Deep"
---

# How do you architect an evidence-generating audit trail so proving 'who accessed which sensitive dataset when, and under what policy' is a query over immutable access logs correlated with the catalog? Where does relying on per-engine native audit logs create gaps a determined auditor will find?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ GDPR Deletion in Immutable Lakes](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/GDPR_Deletion_in_Immutable_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an evidence-generating audit trail that proves 'who accessed which sensitive dataset when, and under what policy,' we need to design a robust data architecture using a combination of Data Lakes, Data Warehouses, and a Data Mesh approach. The key is ensuring immutable access logs are correlated with the catalog in a way that provides comprehensive and verifiable records. Per-engine native audit logs can create gaps, so relying solely on them may not be sufficient for thorough auditing.

First, let's break down the core requirements:

Immutability: Ensure all logs are timestamped and cannot be altered after creation.
Correlation with Catalog: Link each access event to its corresponding dataset in a unified catalog.
Policy Compliance: Track how policies were applied during data accesses.

We will use a Data Mesh architecture, which distributes data ownership across organizational domains while maintaining a central governance layer. This approach allows for granular control and visibility into data access patterns across different environments.

#### Triage
What's Being Probed

The interviewer is probing your understanding of:

Data Architecture: How to design systems that support comprehensive auditing.
Immutability: Techniques for ensuring logs cannot be tampered with.
Correlation: Mechanisms to link audit logs with the data catalog.
Policy Tracking: Methods for logging and enforcing access policies.

#### Assumptions
The system must handle large volumes of log data.
There will be multiple data sources and engines (e.g., relational databases, NoSQL, Hadoop).
Compliance requirements dictate regular audits.
Step-by-Step Solution from First Principles
1. Data Model Design
Audit Log Schema: Define a schema for audit logs that includes fields like timestamp, user_id, dataset_name, access_type (read/write), and policy_id.
SQL
CREATE TABLE AuditLogs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    timestamp TIMESTAMP NOT NULL,
    user_id VARCHAR(255) NOT NULL,
    dataset_name VARCHAR(255) NOT NULL,
    access_type ENUM('READ', 'WRITE') NOT NULL,
    policy_id INT NOT NULL,
    FOREIGN KEY (policy_id) REFERENCES Policies(id)
);
Data Catalog Schema: Define a schema for the data catalog that includes fields like dataset_name, description, and policies_applied.
SQL
CREATE TABLE DataCatalog (
    id INT AUTO_INCREMENT PRIMARY KEY,
    dataset_name VARCHAR(255) NOT NULL UNIQUE,
    description TEXT,
    policies_applied JSON  -- Stores policy details as JSON
);
2. Immutable Logging
Use a distributed log storage system like Apache Kafka or AWS Kinesis to ensure logs are written in real-time and cannot be altered.
Implement event sourcing where each log entry is treated as an immutable transaction.
3. Correlation with Catalog
Integrate the audit log schema with the data catalog through a central metadata service. This ensures that every dataset has its corresponding policies linked to it.
SQL
-- Example of linking policies in the DataCatalog
INSERT INTO DataCatalog (dataset_name, description, policies_applied) VALUES ('SensitiveData', 'Contains personal information.', '[{"id": 1, "name": "ReadAccessPolicy"}]');
4. Policy Tracking
Define a policy schema that includes fields like id, name, and description.
SQL
CREATE TABLE Policies (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL UNIQUE,
    description TEXT
);
Ensure policies are enforced at the database level using appropriate access control mechanisms.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary
By leveraging a Data Mesh architecture, we can ensure that audit logs are immutable and correlated with the data catalog. This approach provides a comprehensive trail of who accessed which sensitive dataset when, and under what policy. Using distributed log storage systems ensures that logs cannot be tampered with, while integrating them into the central metadata service maintains consistency.

#### Common Mistakes

Over-reliance on Per-Engine Audit Logs: Relying solely on per-engine native audit logs can leave gaps in coverage.
Lack of Immutability: Ensuring that audit logs are immutable is crucial; otherwise, they may be tampered with.
Poor Correlation Mechanisms: Not linking audit logs to the data catalog properly can lead to fragmented and incomplete audit records.

#### Follow-Ups

How would you ensure real-time processing of audit logs?
What measures would you take to handle failures in logging mechanisms?
Can you discuss any trade-offs between centralized and decentralized logging approaches?

**Comparison:**

```
FLOWCHART · MERMAID
erDiagram
    AuditLogs ||--o{ Policies : has_policy
    DataCatalog ||--| Policies : is_subject_to
    AuditLogs {
        int id PK
        timestamp timestamp
        varchar(255) user_id
        varchar(255) dataset_name
        enum('READ', 'WRITE') access_type
        int policy_id FK
    }
    DataCatalog {
        int id PK
        varchar(255) dataset_name
        text description
        json policies_applied
    }
    Policies {
        int id PK
        varchar(255) name
        text description
    }
```

**Code:**

```sql
CREATE TABLE AuditLogs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    timestamp TIMESTAMP NOT NULL,
    user_id VARCHAR(255) NOT NULL,
    dataset_name VARCHAR(255) NOT NULL,
    access_type ENUM('READ', 'WRITE') NOT NULL,
    policy_id INT NOT NULL,
    FOREIGN KEY (policy_id) REFERENCES Policies(id)
);

CREATE TABLE DataCatalog (
    id INT AUTO_INCREMENT PRIMARY KEY,
    dataset_name VARCHAR(255) NOT NULL UNIQUE,
    description TEXT,
    policies_applied JSON
);

CREATE TABLE Policies (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL UNIQUE,
    description TEXT
);
```

**Example input / output:**

```text
Input:
User ID: 12345
Timestamp: 2023-10-01T10:00:00Z
Dataset Name: SensitiveData
Access Type: READ

Output:
{
  "timestamp": "2023-10-01T10:00:00Z",
  "user_id": 12345,
  "dataset_name": "SensitiveData",
  "access_type": "READ",
  "policy_id": 1
}
```

**Data:**

```json
{
  "dataset_name": "SensitiveData",
  "description": "Contains personal information.",
  "policies_applied": [
    {
      "id": 1,
      "name": "ReadAccessPolicy"
    }
  ]
}
```

**Flow:**

1. Invalid
2. Valid
3. Failed
4. Success
5. User Access
6. AuditLogWriter
7. Log Validity
8. Reject Log
9. Log Correlation
10. CatalogLookupError
11. PersistLog

