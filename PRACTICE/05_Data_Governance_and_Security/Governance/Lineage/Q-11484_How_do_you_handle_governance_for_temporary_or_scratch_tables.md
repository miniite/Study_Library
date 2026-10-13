---
type: "practice"
difficulty: "Intermediate"
---

# How do you handle governance for temporary or scratch tables that analysts create ad hoc? Do they even belong in the catalog?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Ownership](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Data_Ownership.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with data governance and lineage, it's essential to manage both permanent and temporary or scratch tables effectively. Temporary or scratch tables are often created by analysts during exploratory data analysis (EDA), which can make them a bit tricky to handle in the context of formal data governance frameworks. However, even these tables have value, and their proper management is crucial for maintaining overall data integrity.

TL;DR

Temporary tables should be managed within a data governance framework to ensure consistency, traceability, and regulatory compliance. While they don't need as much oversight as permanent tables, they still warrant some attention through mechanisms like naming conventions, metadata documentation, and periodic reviews.

#### Triage
Restating the Question and What's Being Probed

The question asks about handling temporary or scratch tables used by analysts for ad hoc work in a data governance context. The core of this is understanding how to balance flexibility with governance requirements. The interviewer likely wants to see your approach on:

Identifying these tables.
Managing their creation and usage.
Ensuring compliance and traceability.
Step-by-Step Solution: From First Principles
1. Define the Scope of Temporary Tables

Temporary or scratch tables are often short-lived, used for in-memory processing, or created as a result of ad hoc queries. These might include:

Intermediate results from EDA.
Transient data staging areas.
2. Metadata Management

Even though these tables aren't permanent, they should still be documented and tracked within the governance framework. This includes:

Naming Conventions: Standardize names to indicate their temporary nature (e.g., prefix with "scratch_" or "tmp_").
Documentation: Maintain a metadata repository that tracks when and why each table was created.
Access Control: Ensure that access is restricted to authorized personnel.
3. Catalog Management

Although not typically part of the permanent data catalog, temporary tables can be registered in a separate "scratch" section:

Create a Separate Catalog: Designate a specific area or section within the metadata repository for these tables.
Version Control: For complex pipelines where multiple iterations are common, maintain version control to track changes.
4. Compliance and Traceability

Even ad hoc queries can have significant business implications. To ensure compliance:

Audit Trails: Log all operations on temporary tables to track usage and lineage.
Governance Policies: Establish policies that require approval for the creation of non-standard or complex queries using these tables.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Temporary tables, despite their transient nature, still require careful management. By incorporating them into a broader data governance framework with appropriate metadata tracking and compliance measures, you can maintain both flexibility and control. Common mistakes include:

Overlooking the importance of temporary table documentation.
Failing to establish clear policies for their use.

#### Follow-ups
might include discussing tools or technologies that support these practices (e.g., data catalog services) or how to integrate temporary tables into a broader data lifecycle management strategy.

**Comparison:**

| Name          | Created By  | Purpose                            | Timestamp        |
|---------------|-------------|------------------------------------|------------------|
| tmp_sales_data_2023 | John Doe   | EDA for Q4 sales analysis         | 2023-10-01 15:30 |

**Code:**

```python
# Example code snippet to log operations on temporary tables
def log_temp_table_operation(table_name, operation):
    with open('temp_tables_audit.log', 'a') as f:
        timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        f.write(f"{timestamp} - {operation} performed on table: {table_name}\n")
```

**Example input / output:**

```text
Input:
- Analyst creates a temporary table named 'tmp_sales_data_2023' for EDA.
Output:
- Metadata repository logs the creation of 'tmp_sales_data_2023'.
- Audit log records the timestamp and nature of the operation.
```

**Data:**

```json
{
  "metadata_repository": {
    "scratch_tables": [
      {"name": "tmp_sales_data_2023", "created_by": "John Doe", "purpose": "EDA for Q4 sales analysis", "timestamp": "2023-10-01"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. User creates temporary table
4. Is the table part of a formalprocess?
5. Document and log operations
6. Log operation for traceabilityonly
7. Audit trail created forcompliance
8. Include in metadata repositoryfor tracking

