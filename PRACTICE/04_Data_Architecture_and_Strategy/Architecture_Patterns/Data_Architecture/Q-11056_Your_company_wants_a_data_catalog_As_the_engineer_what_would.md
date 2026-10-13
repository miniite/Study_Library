---
type: "practice"
difficulty: "Intermediate"
---

# Your company wants 'a data catalog.' As the engineer, what would actually go in it, how would you keep it current, and what breaks if you rely on people to update it manually?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distinguishing Drift from Pipeline Breaks](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Distinguishing_Drift_from_Pipeline_Breaks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

A data catalog is a central repository that documents your organization's data assets, including metadata such as column descriptions, data types, lineage, and other details. As an intermediate-level Data Engineer, you need to ensure that the data catalog stays current by automating updates where possible or setting up manual processes with clear guidelines.

#### Triaging the Question

The core of this question revolves around creating a robust data catalog to manage metadata effectively. The interviewer is probing your understanding of:

Metadata Management: How metadata is structured and used.
Automation vs. Manual Updates: Strategies for ensuring metadata stays current without overburdening teams.
Failure Modes: Potential issues with manual updates and how they can be mitigated.

#### First Principles Solution
1. Defining the Data Catalog

A data catalog typically includes:

Column Descriptions: Detailed descriptions of each column in your datasets.
Data Types: Information about the data types (e.g., integer, string).
Lineage: Historical context on how data is transformed and processed.
Schemas: Definitions of data structures.
Tags and Labels: Categorization for easy navigation.
Usage Metrics: How often the data is queried or modified.
2. Automated Metadata Collection

To keep the catalog current, you can leverage automated tools to gather metadata:

ETL Tools: Extract metadata during ETL processes.
Data Catalog APIs: Use APIs provided by cloud services like AWS Glue, Google BigQuery Metadata API, or Azure Data Catalog.
Database Integration: Automate metadata collection from databases using database introspection and schema analysis.
3. Manual Updates

Manual updates are necessary for non-standard datasets or when automated tools fall short. To manage this:

Clear Guidelines: Document a clear process for manual updates, including who is responsible and the frequency.
Version Control: Keep track of changes made manually to maintain auditability.
Notifications: Set up alerts when data changes significantly, so manual updates can be prioritized.
4. Data Quality Checks

Implement data quality checks to ensure the catalog remains accurate:

Schema Validation: Validate incoming data against defined schemas.
Data Integrity Rules: Ensure data meets certain quality standards before being added or updated in the catalog.
Regular Audits: Conduct periodic audits of the data catalog to verify its accuracy.
5. Data Lineage Tracking

To understand how data changes over time:

Event Logging: Track every change made to datasets, including who made the changes and when.
Transformation Logs: Log transformations applied to raw data during ETL processes.
Change Management Tools: Use tools like Change Data Capture (CDC) to track modifications.
6. User Interface

Provide a user-friendly interface for accessing the catalog:

Search Functionality: Allow users to search for datasets based on tags, names, and descriptions.
Permissions Management: Implement role-based access control to ensure sensitive data is protected.
Documentation: Provide documentation and support materials to help users understand how to use the catalog.

#### Closing Summary

By combining automated metadata collection with well-defined manual update processes, you can maintain a robust data catalog. Automated tools reduce the burden on teams while ensuring that critical datasets are always up-to-date. However, it's crucial to have clear guidelines for manual updates and implement checks to prevent errors or inconsistencies.

#### Common Mistakes
and Follow-Ups
Overreliance on Manual Updates: Relying too much on manual processes can lead to inconsistent metadata and increased workload.
Lack of Automation: Failing to automate metadata collection from tools like ETL platforms or databases can result in outdated catalogs.
Poor Data Quality Checks: Insufficient data quality checks can lead to inaccurate catalog entries, affecting downstream processes.

#### Follow-Ups
How would you handle changes in schema for a large dataset?
What are the trade-offs between manual and automated updates?
Can you walk through an example of how you've implemented a similar system in your previous role?

**Comparison:**

| Database Name | Table Name | Column | Data Type |
|---------------|------------|--------|----------|
| SalesDB       | customers  | id     | integer  |
|               |            | name   | string   |
|               |            | email  | string   |
| Database Name | Table Name         | Description                                    |
|---------------|--------------------|------------------------------------------------|
| SalesDB       | customer_preferences | Customer preferences and purchase history.    |

**Code:**

```python
# Example of automating metadata collection using AWS Glue
import boto3

def get_glue_tables(database_name):
    client = boto3.client('glue')
    response = client.get_tables(DatabaseName=database_name)
    return response['TableList']

tables = get_glue_tables('your_database')
for table in tables:
    print(table['Name'], table['StorageDescriptor'])
```

**Example input / output:**

```text
Input:
- A user queries the data catalog for a table named 'sales_data'.

Output:
Metadata for 'sales_data':
- Columns: product_id, customer_id, amount
- Description: Monthly sales data for all products
- Lineage: Raw data filtered and aggregated to form monthly sales reports.
```

**Data:**

```json
{
  "metadata": {
    "table_name": "sales_data",
    "columns": [
      { "name": "product_id", "type": "integer" },
      { "name": "customer_id", "type": "string" },
      { "name": "amount", "type": "float" }
    ],
    "description": "Monthly sales data for all products",
    "lineage": [
      {
        "source_tables": ["raw_sales_data"],
        "transformations": ["filter", "aggregate"],
        "target_table": "monthly_sales"
      }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Collection
4. ETL Tools
5. Automated Updates
6. Catalog Update
7. Manual Entry Process
8. Version Control
9. Notifications System

