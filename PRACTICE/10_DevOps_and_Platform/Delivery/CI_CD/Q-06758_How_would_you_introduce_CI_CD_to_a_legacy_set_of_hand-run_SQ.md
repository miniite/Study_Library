---
type: "practice"
difficulty: "Intermediate"
---

# How would you introduce CI/CD to a legacy set of hand-run SQL scripts and cron'd Spark jobs with no tests; what's the first thing you automate to build a safety net before touching the deploy process?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Introducing CI CD to Legacy Systems](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Introducing_CI_CD_to_Legacy_Systems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When introducing Continuous Integration and Continuous Deployment (CI/CD) to a legacy system involving hand-run SQL scripts and cron'd Spark jobs, the primary goal is to gradually build a robust testing framework that can catch any regression or issues introduced during development. The first thing you should automate is a suite of unit tests for your SQL scripts and integration tests for your Spark jobs. This will serve as your initial safety net before you start automating the deployment process.

To triage this question, we need to understand what CI/CD means in practice. CI/CD involves automatically running code through a series of checks and deployments, ensuring that every change is reliable and can be deployed without breaking existing functionality. The legacy system currently relies on manual execution, which poses significant risks. By automating tests first, you establish a baseline for acceptable behavior.

#### Assumptions

We are dealing with Python or Scala-based Spark jobs.
SQL scripts are written in a dialect supported by a database like PostgreSQL or MySQL.
There is an existing codebase for the Spark jobs and some understanding of their logic.
The database schema can be introspected programmatically.

Let's reason through the process step-by-step:

Identify Testable Components:

SQL Scripts: These are responsible for data transformations, updates, or queries.
Spark Jobs: These handle large-scale data processing tasks and may involve reading from databases, performing complex computations, and writing results back.

Design Unit Tests for SQL Scripts:

Mock Databases: Use tools like SQLite in-memory databases to mimic production environments without affecting real data.
Test Cases: Write test cases that validate the integrity of the transformations or queries. For example, if a script is supposed to aggregate sales data, write tests to ensure it handles edge cases (e.g., null values, large datasets).

Design Integration Tests for Spark Jobs:

Data Ingestion: Test how your jobs handle different types of input data.
Process Validation: Ensure that the processing logic is correct and consistent with expected outputs.
Error Handling: Check how your jobs behave under unexpected conditions, such as network failures or partial data availability.

Set Up an Automated Build Process:

Version Control Repository: Use Git to manage source code versions.
CI Server Setup: Configure a CI server like Jenkins, GitHub Actions, or GitLab CI/CD to automatically run tests on every commit.
Build Steps: Define steps for running unit tests and integration tests. Ensure that these steps are idempotent and can be executed in any environment.

Automate the Deployment Process:

Containerization: Use Docker to package your Spark jobs and SQL scripts into container images.
Orchestration Tools: Employ Kubernetes or another orchestration tool to manage deployments across environments (development, staging, production).
Rollback Mechanisms: Implement rollback strategies in case a deployment fails.

By following these steps, you create a safety net that helps prevent regressions and ensures that any changes are thoroughly tested before being deployed. This approach is crucial for maintaining stability as new features are added to the system.

#### Closing Summary
The first step in introducing CI/CD to this legacy system is automating unit tests for SQL scripts and integration tests for Spark jobs. This provides a reliable baseline for validating behavior and catching issues early. Gradually, you can extend this coverage to include more comprehensive testing and automated deployments, ensuring that the transition to CI/CD is smooth and safe.

#### Common Mistakes

Failing to identify testable components and underestimating the complexity of writing robust tests.
Skipping setup steps for version control and CI servers, leading to manual or ad-hoc testing processes.
Overlooking rollback mechanisms, which can lead to significant downtime if a deployment fails.

**Comparison:**

| Component | Description |
|-----------|-------------|
| Unit Tests | Validate individual scripts for correctness. |
| Integration Tests | Ensure jobs process data correctly and handle edge cases. |
| Build Steps | Run tests on every commit using CI tools like Jenkins or GitHub Actions. |

**Code:**

```
Example Test Case for an SQL Script using pytest

def test_aggregate_sales_data(): # Arrange from your_sql_script import aggregate_sales

CODE
Copy
mock_db = SQLiteMemoryDatabase()  # In-memory database setup
input_data = load_mock_data()

# Act
result = aggregate_sales(mock_db, input_data)

# Assert
assert result == expected_aggregated_result
CODE
Copy

```python
# Example Integration Test for a Spark Job using PySpark
from pyspark.sql import SparkSession

def test_spark_job_integration():
    spark = SparkSession.builder.appName("TestJob").getOrCreate()
    input_df = load_test_data()  # Load test data

    # Act
    output_df = your_spark_job(input_df)

    # Assert
    assert output_df.count() == expected_count
```

**Example input / output:**

```text
Input:
- SQL Script input data: [{"id": 1, "sales": 100}, {"id": 2, "sales": 50}]
- Spark Job input data: [{"id": 1, "value": 200}, {"id": 2, "value": 300}]

Output:
- SQL Script result: 150
- Spark Job output count: 2
```

**Data:**

```json
{
  "test_sql_script": {
    "input_data": [
      {"id": 1, "sales": 100},
      {"id": 2, "sales": 50}
    ],
    "expected_aggregated_result": 150
  },
  "test_spark_job": {
    "input_data": [
      {"id": 1, "value": 200},
      {"id": 2, "value": 300}
    ],
    "expected_count": 2
  }
}
```

**Flow:**

1. Version Control
2. CI Server Setup
3. Test Automation
4. Automated Deployments
5. Rollback Mechanisms

