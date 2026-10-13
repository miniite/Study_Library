---
type: "practice"
difficulty: "Deep"
---

# You inherit a pipeline with zero tests and constant incidents. What's your first-90-days plan to instrument data quality without boiling the ocean?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Validation Coverage and Quarantine](<../../../../02_Data_Engineering/Data_Quality/Validation/Validation_Coverage_and_Quarantine.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Inheriting a pipeline with no existing data quality or validation mechanisms is a critical challenge that requires a structured, yet pragmatic approach. The primary goal in the first 90 days is to establish a robust and scalable system of checks to prevent issues from escalating while minimizing disruption to ongoing operations. Here’s how I would tackle this:

Firstly, it's crucial to understand the current state of the pipeline by gathering data about its historical performance. This involves identifying key metrics such as throughput, error rates, and latency. Understanding the nature of the incidents will help in prioritizing which areas need immediate attention.

The initial steps involve defining the core elements that require validation. These typically include data completeness, consistency, format correctness, and adherence to business rules. Establishing a framework for testing these critical aspects is essential before diving into more complex validations.

Let’s break down the first 90 days plan step by step:

Assessment & Baseline Setup: Spend the first week understanding the pipeline architecture, data sources, transformations, and destinations. Define key performance indicators (KPIs) that will help in monitoring the health of the data.
Critical Path Identification: Identify critical paths where failure can have a significant impact on downstream processes or business outcomes. This could be areas with high transaction volumes, complex transformations, or sensitive data handling.
Incremental Testing Strategy: Start by instrumenting tests that are easy to implement but provide maximum value. For example, basic schema validations and integrity checks.
Automated Testing Frameworks: Leverage tools like Apache Beam, Apache Flink, or AWS Glue for implementing automated testing. These tools can help in setting up continuous integration/continuous deployment (CI/CD) pipelines that run tests regularly.
Monitoring & Alerting Systems: Integrate monitoring and alerting mechanisms using solutions like Prometheus, Grafana, or CloudWatch to track real-time data quality metrics and trigger alerts for anomalies.
Reporting & Visualization: Create dashboards that provide insights into the health of the pipeline. Use tools like Tableau, Looker, or Power BI to visualize key metrics and trends over time.

**Comparison:**

| Data Quality Metrics | Current Status | Target | Improvement Strategy |
|----------------------|----------------|--------|----------------------|
| Completeness         | Low            | High   | Implement row-level validation |
| Consistency          | Medium         | High   | Integrate field-level checks |
| Format Correctness   | Medium         | High   | Use regular expressions or schema rules |
| Business Rule Adherence | Low      | High   | Incorporate domain-specific validations |

**Code:**

```python
# Example of a basic schema validation function in Python using PySpark
def validate_schema(df):
    expected_schema = StructType([
        StructField("id", IntegerType(), True),
        StructField("name", StringType(), True)
    ])
    actual_schema = df.schema
    if not actual_schema == expected_schema:
        raise ValueError("Schema does not match expected schema")
```

**Example input / output:**

```
Input:
JSON
Copy
[
  {"id": 1, "name": "John Doe"},
  {"id": null, "name": "Jane Smith"}
]
Output:
PLAINTEXT
Copy
Schema validation failed for the second record. The 'id' field cannot be null.
```

**Data:**

```json
{
  "data": [
    {
      "id": 1,
      "name": "John Doe"
    },
    {
      "id": null, // Expected to fail validation
      "name": "Jane Smith"
    }
  ]
}
```

**Flow:**

1. Inherit Pipeline
2. Identify Critical Paths
3. Define Core Validation Tests
4. Implement Automated Tests
5. Integrate Monitoring &Alerting
6. Create Dashboards forReporting

