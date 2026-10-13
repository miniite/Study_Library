---
type: "practice"
difficulty: "Expert"
---

# You need column-level lineage across SQL, Spark, dbt, and hand-written Python jobs. Where does automated lineage extraction break down, and how do you close the gaps without asking every team to annotate manually?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scaling Lineage Graphs](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Scaling_Lineage_Graphs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Automated lineage extraction can face challenges when dealing with diverse data processing systems like SQL, Spark, dbt, and hand-written Python jobs. To address these gaps without manual annotations from teams, we need to leverage a combination of static code analysis, metadata capture during execution, and dynamic tracking.

#### Triage
The question is about ensuring column-level lineage across multiple data processing environments (SQL, Spark, dbt, and Python) with an emphasis on automated extraction. The key points the interviewer might be probing include understanding the limitations of automatic lineage tools, how to mitigate these issues, and what strategies can be employed to maintain robust metadata without excessive manual effort.

#### First-Principles Solution

Static Code Analysis: Tools like bandit for Python or static analysis in Spark jobs can capture initial dependencies.
Metadata Capture During Execution: Leverage the execution environment to log lineage information as data flows through processes.
Dynamic Tracking and Event Logging: Use event-driven architectures to track changes and updates in real-time.

#### Implementation
Steps:

Identify Tools for Static Analysis:

For Python, use bandit or custom linters that can parse scripts to detect potential dependencies.
For Spark jobs, utilize the SparkSession API to capture lineage information during job execution.

Metadata Extraction During Execution:

Integrate logging mechanisms in SQL and dbt pipelines to log references between datasets.
Use Spark's built-in features like SparkConf or custom UDFs to track lineage within the codebase.

Dynamic Tracking with Event Logging:

Implement event-driven architectures using tools like Apache Kafka, where data movement events are logged in a central metadata repository.
Set up monitoring and alerting systems to notify of changes in lineage that require manual intervention or review.

Automation Frameworks and Integrations:

Use frameworks like great expectations for Python to ensure consistent lineage logging across different environments.
Integrate with CI/CD pipelines to automatically capture lineage information during builds and deployments.

Centralized Metadata Repository:

Establish a central metadata repository that stores lineage data, ensuring consistency and accessibility.
Use APIs like Apache Atlas or custom-built solutions to query and visualize the lineage information.

#### Closing Summary
+ Common Mistakes/Follow-Ups: By leveraging static analysis, real-time logging, and event-driven architectures, we can automate much of the lineage extraction process. However, manual annotations might still be necessary for complex or legacy systems. To avoid common pitfalls:

Ensure Consistency Across Tools: Different tools may have varying levels of support for capturing lineage.
Regularly Review Lineage Logs: Manual reviews and updates are crucial to maintain accuracy in dynamic environments.
Monitor Performance Impact: Automated logging should not significantly impact performance, especially during high-frequency data processing.

#### Common Mistakes

Over-reliance on manual annotations without proper automation infrastructure.
Ignoring dynamic changes or real-time events that can disrupt lineage tracking.
Inadequate testing of automated tools in production environments.

#### Follow-Ups

How would you handle legacy systems with no logging capabilities?
What are the trade-offs between automated and manual approaches to lineage capture?
Can you provide an example of a tool or framework that integrates well with your proposed solution?

**Comparison:**

```
Source Column	Target Column	Transformation
age	customer_age	Derived from input data
name	customer_name	Direct mapping
```

**Code:**

```
Example Python code using bandit for static analysis

import bandit

def parse_dependencies(script): findings = [] parser = bandit.Parser() results = parser.parse_file(script) for result in results: if result.severity >= 3: findings.append(result.line) return findings

CODE
Copy

```python
# Example Python code using great expectations for lineage logging
import great_expectations as ge

df = ge.read_csv("path/to/csv")
df.expect_column_values_to_be_between("column_name", min_value=0, max_value=100)
```

**Example input / output:**

```text
Input:
- A Spark job with dependencies on multiple tables and transformations.

Output:
- Metadata capture in a central repository, including lineage information for each column transformation.
```

**Data:**

```
Sample Data:
JSON
Copy
{
  "job_id": "spark-job-001",
  "source_table": "customer_data",
  "target_table": "processed_customers",
  "columns": [
    {"source_column": "age", "target_column": "customer_age"},
    {"source_column": "name", "target_column": "customer_name"}
  ]
}
Metadata Schema:
JSON
Copy
{
  "job_id": "",
  "environment": "dev",
  "timestamp": "",
  "input_tables": [],
  "output_tables": [],
  "columns_map": []
}
```

**Flow:**

1. Yes
2. No
3. Static Analysis
4. Is Static Tool Sufficient?
5. Log Lineage During Execution
6. Implement Dynamic Tracking
7. Central Metadata Repository
8. Monitor and Integrate withCI/CD

