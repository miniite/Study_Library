---
type: "practice"
difficulty: "Deep"
---

# When you say 'golden record', what is the actual data model and lineage behind it — do you materialize one physical golden record, keep source-attributed survivorship, or expose a virtual golden view, and what drove that choice?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Golden Records and Survivorship](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Golden_Records_and_Survivorship.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing "golden records" in Master Data Management (MDM), we're talking about a canonical representation of an entity that consolidates data from multiple sources into a single, trusted version. The actual data model and lineage behind it can vary significantly based on the requirements, scale, and organization's MDM strategy. Here, I'll explore three common approaches: materializing one physical golden record, keeping source-attributed survivorship, and exposing a virtual golden view. Each approach has its trade-offs and is driven by different business needs.

Firstly, let's define the terms:

Physical Golden Record: A single, unified database or table that contains the canonical data for an entity.
Source-Attributed Survivorship: Storing multiple versions of the same record but tagging them with their source origin to maintain historical context.
Virtual Golden View: Using a view or query to construct a golden record on-the-fly without physically storing it.

The choice between these approaches is driven by factors such as performance, data governance, and the organization's MDM strategy. Now, let's break down each approach in detail:

Physical Golden Record

Data Model:

A single table or database that serves as the source of truth.
Each record includes a canonical version of all attributes for an entity.

Lineage:

Maintains a clear lineage by storing historical versions and updates through transactions.
Tracks changes with timestamps, audit logs, etc., to ensure data integrity and traceability.

#### Pros

Simplified queries due to the single source of truth.
Easier to enforce consistency rules and governance policies.
Suitable for scenarios where real-time access to a consistent view is critical.

#### Cons

Increased complexity in managing updates across multiple sources.
Storage overhead as each entity can have multiple versions over time.
Potential performance issues with large datasets if not optimized properly.
Source-Attributed Survivorship

Data Model:

Multiple physical records for the same entity, each tagged with its source of origin.
A metadata table or attribute to store lineage and versioning information.

Lineage:

Preserves historical context by storing multiple versions.
Allows for detailed auditing and data lineage tracking without sacrificing consistency at a point in time.

#### Pros

Retains the ability to trace back to original sources, enhancing data integrity.
Supports more robust compliance requirements where historical data is critical.
Simplifies incremental updates as changes are applied to the most recent version.

#### Cons

Complex queries and joins required to consolidate records.
Increased storage overhead due to multiple versions of each record.
Potential performance issues with large datasets and frequent updates.
Virtual Golden View

Data Model:

No physical golden record; a virtual view constructed on-demand using SQL or other query languages.
Data is sourced from multiple tables and combined in real-time.

Lineage:

Dynamically generated queries that reflect the most up-to-date data across sources.
Challenges in maintaining consistent updates due to dynamic nature.

#### Pros

Minimal storage overhead as no physical records are stored.
High scalability and performance as data is fetched on-demand.
Flexibility in updating the view without altering underlying data structures.

#### Cons

Complex queries can be resource-intensive, especially under high load.
Risk of inconsistent or stale data if not properly managed.
Limited support for complex transactions and updates.
Driving Factors

The choice between these approaches is driven by various factors:

Data Volume: Larger datasets may favor source-attributed survivorship or virtual views to manage storage efficiently.
Performance Requirements: Real-time access often favors a physical golden record, while batch processing might benefit from a more flexible approach like virtual views.
Governance and Compliance: Organizations with stringent compliance requirements might prefer detailed lineage tracking provided by source-attributed survivorship.
Resource Constraints: Limited storage or computational resources might necessitate the use of virtual views.

In summary, the actual data model and lineage behind a golden record can significantly vary based on the specific needs of an organization. Each approach has its strengths and weaknesses, making it crucial to align the chosen method with business goals and technical constraints.

**Comparison:**

| ID | Source System | Attribute 1 | Attribute 2 | Latest Version       |
|----|---------------|-------------|-------------|----------------------|
| 1001 | SystemA       | Value1      | null        | 2023-10-01T14:00:00Z |
| 1001 | SystemB       | null        | Value2      | 2023-10-01T15:00:00Z |

**Code:**

```sql
-- Example SQL for constructing a virtual golden view

SELECT
    id,
    source_system,
    attribute1,
    attribute2,
    -- ... other attributes
    max(updated_at) OVER (PARTITION BY id) AS latest_version
FROM
    source_table
JOIN
    another_source ON source_table.id = another_source.id
```

**Example input / output:**

```text
Input:
source_table = {
  id: 1001,
  source_system: "SystemA",
  attribute1: "Value1",
  updated_at: "2023-10-01T14:00:00Z"
}

another_source = {
  id: 1001,
  source_system: "SystemB",
  attribute2: "Value2",
  updated_at: "2023-10-01T15:00:00Z"
}

Output:
virtual_golden_view = {
  id: 1001,
  source_system: "SystemB",   -- Latest update from SystemB
  attribute1: "Value1",
  attribute2: "Value2",
  latest_version: "2023-10-01T15:00:00Z"
}
```

**Data:**

```json
{
  "id": 1001,
  "source_system": "SystemA",
  "attribute1": "Value1",
  "attribute2": "Value2",
  "latest_version": "2023-10-01T14:30:00Z"
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Source Table
6. Is latest update?
7. Update Virtual View
8. Keep existing view
9. Virtual View Updated
10. Another Source Table
11. Is latest update?
12. Keep existing view
13. Update Virtual View
14. Virtual Golden ViewConstructed
15. Virtual Golden ViewConstructed

