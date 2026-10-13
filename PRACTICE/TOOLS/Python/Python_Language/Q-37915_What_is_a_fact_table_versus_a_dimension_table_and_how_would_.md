---
type: "practice"
difficulty: "Intermediate"
---

# What is a fact table versus a dimension table, and how would a Python load job treat each differently in terms of update frequency and grain?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling Schema Evolution in Python](<../../../../TOOLS/Python/Python_Language/Handling_Schema_Evolution_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Fact tables and dimension tables are key components in data warehousing and business intelligence systems, typically used to store transactional and descriptive data respectively. For a Python load job, understanding how each type of table is treated differently—especially concerning update frequency and grain—is crucial for efficient data management.

Fact tables capture the actual business events or transactions, often at a high granularity, while dimension tables provide detailed descriptions of these events. In terms of a Python ETL (Extract, Transform, Load) job, you would handle these two types of tables with distinct strategies due to their inherent differences in size and update frequency.

Firstly, let’s break down the key characteristics:

Granularity: Fact tables have finer granularity, typically recording each transaction or event. Dimension tables are coarser-grained, holding descriptive attributes that provide context.
Update Frequency: Fact tables are generally updated more frequently as new transactions occur. Dimension tables are less dynamic and often updated based on changes in business rules or external data.

#### Triaging the Question

The question is asking for an explanation of how Python ETL jobs would handle fact tables versus dimension tables with respect to their update frequency and grain. The interviewer is likely probing your understanding of data warehousing principles, ETL processes, and best practices for managing different types of data.

#### First-Principles Solution
Fact Tables

Characteristics: High-granularity records representing transactions or events. Update Frequency: Frequent updates as new transactions occur. Handling in Python:

Incremental Loading: Use incremental loading techniques to only update rows that have changed since the last load. This can be done by comparing a timestamp column with the current date.
Hash Join: For large datasets, use hash joins or other efficient algorithms to match records from dimension tables based on keys. This ensures accurate and timely updates without needing full reprocessing.
PYTHON
# Example: Incremental loading of fact table data
import pandas as pd

def load_fact_table(fact_df, dimensions):
    # Assuming `fact_df` contains a timestamp column 'last_update'
    recent_updates = fact_df[fact_df['last_update'] >= last_load_time]

    # Perform a hash join to update only new or updated records
    updated_records = recent_updates.merge(dimensions, on='dimension_key', how='left')

    return updated_records

# Example usage:
last_load_time = pd.Timestamp('2023-10-01 00:00:00')
fact_df = pd.read_csv('path/to/fact_table.csv')
dimensions = pd.read_csv('path/to/dimensions.csv')

updated_fact_data = load_fact_table(fact_df, dimensions)
Dimension Tables

Characteristics: Coarse-grained records holding descriptive attributes. Update Frequency: Less frequent updates based on business rules or external data sources. Handling in Python:

Full Refresh: Typically require a full refresh when changes occur. This involves reprocessing the entire table, but can be optimized by using versioning or snapshot techniques.
Materialized Views: Use materialized views to store precomputed summaries, reducing read times during analysis.
PYTHON
# Example: Full refresh of dimension table data
def load_dimension_table(dimensions_df):
    # Assuming `dimensions_df` is the updated dataset from an external source
    dimensions_df.to_csv('path/to/dimensions.csv', index=False)

# Example usage:
new_dimensions = pd.read_csv('path/to/new_dimensions.csv')
load_dimension_table(new_dimensions)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, fact tables require more frequent updates with higher granularity, handled through incremental loading techniques. Dimension tables are updated less frequently but need to be processed comprehensively when changes occur.

#### Common Mistakes

Overlooking the need for incremental updates in fact tables, leading to unnecessary full table scans.
Failing to consider the business context that drives update frequency in dimension tables, resulting in inefficient and inflexible data management practices.

#### Follow-up
questions might include discussions on how to handle large datasets efficiently or strategies for maintaining consistency between fact and dimension tables. Understanding these nuances can significantly impact the performance and accuracy of your ETL processes.

**Comparison:**

```
Fact Table	Dimension Table
High-granularity transactions/events	Descriptive attributes for context
Frequent updates (transactional)	Less frequent updates (business rule-driven)
Incremental loading techniques	Full refresh or materialized views
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Fact Table
8. Incremental Update?
9. Update Only Changed Rows
10. Full Refresh
11. Dimension Table
12. Full Update Needed?
13. Full Refresh with MaterializedViews
14. Incremental Updates?
15. Update Selected Rows
16. Full Refresh

