---
type: "practice"
difficulty: "Deep"
---

# How would you approach validating data quality on a very large table where scanning every row is expensive?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Validating Loads and Deduplication](<../../../../02_Data_Engineering/Data_Quality/Validation/Validating_Loads_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To validate data quality on a very large table efficiently, we need to strike a balance between thoroughness and cost-effectiveness. The goal is to ensure that our validation process is both effective and scalable while minimizing the computational resources required. Here’s a step-by-step approach:

Understand the Table Structure: Begin by understanding the schema of the table, including data types, constraints, and any business rules or key business metrics.
Identify Critical Columns: Determine which columns are most critical for your use case. Focus on these as they are likely to have a significant impact on downstream processes.
Sample Data Strategy: Instead of scanning every row, use a sampling technique that provides a representative subset of the data. This helps in identifying common issues without overwhelming resources.

Let's break this down further:

#### Triaging the Question

The question is essentially asking how to validate large datasets efficiently when direct full-table scans are impractical. The interviewer might be probing your understanding of:

Sampling techniques and their trade-offs.
Techniques for scalable validation.
How to handle partial validations that still provide meaningful insights.

#### Assumptions
include:

We have access to the schema and can design queries accordingly.
The data is stored in a database system (e.g., SQL, NoSQL).
There are constraints on computational resources due to cost or performance limitations.

#### Solution

Data Sampling:

Use statistical sampling methods such as simple random sampling, systematic sampling, or stratified sampling to create a representative subset of the data.
For example, if you have 1 million rows, consider using 5% (50,000 rows) for initial validation.

Column-Level Validation:

Validate critical columns first. This could involve checking constraints like unique keys, foreign key relationships, non-null values, and data type consistency.
Use aggregate functions to quickly identify any outlying issues across the sampled data (e.g., count of nulls per column).

Incremental Validation:

Implement incremental validation where new data is periodically checked against existing constraints.
Use change data capture (CDC) logs if your system supports it, allowing you to validate only changes rather than full table scans.

Automated Tools and Scripts:

Develop or use existing ETL tools that can run these validations automatically and log any issues.
Schedule regular validation runs during off-peak hours to minimize impact on performance.

Monitoring and Alerting:

Set up monitoring for the sampled data to detect anomalies in real-time.
Define alert thresholds based on historical data to trigger further investigation if necessary.

#### Closing Summary

By employing a combination of sampling, column-level validation, incremental checks, and automated tools, we can validate large datasets efficiently without scanning every row. This approach ensures that critical issues are identified while keeping the computational costs manageable.

#### Common Mistakes/Follow-Ups
Not Considering Stratified Sampling: Always stratify based on business logic to ensure representative sampling.
Ignoring Incremental Validations: Failing to implement incremental validations can lead to missed changes if data is updated frequently.
Over-Simplifying Sample Size: Choosing a sample size that is too small might not capture all issues, while choosing one that is too large can be resource-intensive.

Would you like to dive deeper into any of these areas or explore additional validation techniques?

**Comparison:**

```
Column	Type	Nullable?
critical_column_1	int	No
critical_column_2	float	Yes
Validation Results
Invalid type for value in row {critical_column: 1, value: 10}
Null value detected in non-nullable column value: {critical_column: 2, value: null}
```

**Code:**

```python
# Example Python code for stratified sampling and column validation

import random

def stratified_sampling(table_data, sample_size):
    # Assuming table_data is a list of dictionaries representing rows
    sampled_rows = []
    strata = {}

    # Stratify based on critical columns (e.g., date or category)
    for row in table_data:
        key = row['critical_column']
        if key not in strata:
            strata[key] = {'count': 0, 'rows': []}
        strata[key]['count'] += 1
        strata[key]['rows'].append(row)

    # Sample from each stratum proportionally
    for _, group_info in strata.items():
        sample_count = int(group_info['count'] * (sample_size / len(table_data)))
        sampled_rows.extend(random.sample(group_info['rows'], sample_count))

    return sampled_rows

def validate_column(row, column_name):
    value = row[column_name]
    if not isinstance(value, expected_type):  # Example type check
        print(f"Invalid type for {column_name} in row: {row}")
    if value is None and not allow_nulls:
        print(f"Null value detected in non-nullable column {column_name}: {row}")

def validate_table(sampled_rows):
    for row in sampled_rows:
        validate_column(row, 'critical_column_1')
        validate_column(row, 'critical_column_2')

# Example usage
table_data = [{'critical_column': 1, 'value': 10}, {'critical_column': 2, 'value': None}, ...]
sampled_data = stratified_sampling(table_data, sample_size=500)
validate_table(sampled_data)
```

**Example input / output:**

```

Input:

table_data: A list of dictionaries representing rows in the table.
sample_size: The number of rows to sample.

Output:

sampled_rows: A subset of rows used for validation.
Validation results logged as messages or stored in a log file.

Example input->output:

PYTHON
Copy
input: {
  "table_data": [
    {"critical_column": 1, "value": 10},
    {"critical_column": 2, "value": null},
    ...
  ],
  "sample_size": 500
}

Output:

CODE
Copy
Invalid type for value in row: {'critical_column': 1, 'value': 10}
Null value detected in non-nullable column value: {'critical_column': 2, 'value': null}
```

**Data:**

```json
{
  "table_data": [
    {"critical_column": 1, "value": 10},
    {"critical_column": 2, "value": null},
    ...
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is data critical?
5. Sampling strategy
6. Select sample size and stratify
7. Validate sampled rows
8. Log results
9. Skip validation for less criticalcolumns
10. End

