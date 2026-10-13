---
type: "practice"
difficulty: "Deep"
---

# An interviewer probes your talent strategy: unified platforms need engineers fluent in streaming semantics, which are scarce. How do you de-risk the platform against a bus-factor of two experts?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Unified Batch and Streaming Platforms](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of creating a unified platform that leverages both batch and stream processing while mitigating the risk associated with a bus-factor of two subject matter experts, we need to focus on several key areas: ensuring comprehensive documentation, fostering knowledge sharing, implementing robust testing practices, and leveraging automation. The primary goal is to make the system understandable and maintainable for new or existing engineers.

The question is essentially asking how to reduce dependency risk by making sure that critical knowledge about streaming semantics is widely distributed within the team. This involves understanding what streaming semantics are (like exactly-once processing, low latency requirements, etc.) and ensuring these principles are embedded in our platform design and documentation.

First-principles reasoning tells us that we need to break down complex concepts into simpler, more understandable components. We should start by documenting all aspects of the system, from architecture decisions to implementation details, ensuring this knowledge is accessible to everyone on the team. This includes not only code but also high-level architectural diagrams and design patterns.

To ensure robustness, we will implement automated tests that cover both functional correctness and edge cases specific to streaming data. We can use frameworks like Apache Flink’s built-in test capabilities or custom test harnesses to validate our stream processing logic comprehensively.

Here is a step-by-step approach:

Documentation: Create comprehensive documentation for all components of the system, focusing on streaming principles such as exactly-once processing and state management.
Training Programs: Develop training sessions that cover not only the technical aspects but also the business implications of using streams effectively.
Code Reviews: Implement rigorous code reviews to ensure consistency in implementation and adherence to best practices.
Knowledge Sharing Sessions: Regularly conduct knowledge-sharing workshops where experts can share their insights with other team members.

**Comparison:**

```
Operation	Description
CreateTableSource	Defines the source for streaming data.
CreateTableSink	Specifies where processed results are stored.
SQLQuery	Defines the processing logic as a table function.
TestSetupAndVerification	Sets up expectations and verifies output against actual results.
```

**Code:**

```python
# Example of an automated test using Apache Flink's testing framework
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, TableConfig

def test_stream_processing():
    env = ExecutionEnvironment.get_execution_environment()
    t_config = TableConfig()
    t_env = StreamTableEnvironment.create(env, t_config)

    # Define and register table source/sinks
    t_env.execute_sql("CREATE TABLE MySource (id INT) WITH (...)")
    t_env.execute_sql("CREATE TABLE MySink (...")

    # Define the processing logic as a table function
    result_table = t_env.sql_query("""
        SELECT id FROM MySource
        GROUP BY id
        EMIT CHANGES
    """)

    # Test setup and verification of expected results
    # This would involve setting up expectations and asserting against actual output
```

**Example input / output:**

```text
Input: [{"id": 1}, {"id": 2}, {"id": 3}]
Expected Output: [(1, 1), (2, 2), (3, 3)]
```

**Data:**

```json
{
  "schema": {
    "id": "INT"
  },
  "sample_data": [
    {"id": 1},
    {"id": 2},
    {"id": 3}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Ingest
4. Is Data Streaming?
5. Process with Flink
6. Batch Processing
7. Output to Storage
8. Output to Storage

