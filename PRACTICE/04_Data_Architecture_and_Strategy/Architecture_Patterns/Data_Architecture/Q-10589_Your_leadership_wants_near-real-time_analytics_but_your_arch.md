---
type: "practice"
difficulty: "Expert"
---

# Your leadership wants near-real-time analytics but your architecture is nightly batch. Sequence the incremental evolution to streaming without a big-bang rewrite. Where do you start and what stays batch forever?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Streaming-first Data Architecture](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Streaming-first_Data_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To evolve our existing nightly batch data processing architecture into one that supports near-real-time analytics, we need to strategically introduce incremental changes while ensuring continuity of service and minimizing disruptions. We'll start by assessing the current architecture, identifying critical components, and then progressively shifting towards a streaming setup.

#### Triaging the Question

What the question is really asking: The leadership wants to achieve real-time or near-real-time analytics, but our existing system processes data on a nightly batch basis. The goal is to incrementally transform this architecture without a significant overhaul (big-bang rewrite).

What is being probed:

Understanding of current systems and their limitations.
Knowledge of streaming architectures and data processing paradigms.
Ability to balance incremental changes with maintaining service continuity.

#### Assumptions
The current nightly batch system is functional but needs improvement for real-time analytics.
There are existing data warehouses or lakes storing historical data.
The team has some experience in both batch and streaming technologies.
The company can afford the time and resources to implement incremental changes.

#### First-Principles Solution

Assess Current Architecture:

Identify critical queries and use cases that require real-time insights.
Evaluate data volume, frequency of updates, and complexity of the processing logic.

Introduce Incremental Changes:

Start by setting up a small-scale proof-of-concept (PoC) to test streaming capabilities.
Gradually integrate streaming for critical paths while keeping others batch.

Build a Streaming Layer:

Use Apache Kafka or AWS Kinesis as a message broker to capture real-time data.
Develop a pipeline to process this stream of events using frameworks like Apache Spark Streaming, Flink, or KStreams.

Merge Batch and Stream Processing:

Integrate the streaming layer with existing batch processing by leveraging techniques such as:
Watermarking: Ensure event time semantics in streams.
Event Time Joining: Combine historical data from the warehouse with real-time events.
Micro-batch Processing: Use smaller batches to balance between stream and batch processing.

Incremental Refactoring:

Refactor critical parts of the application incrementally, starting with less impactful changes.
Monitor performance and adjust as needed.

Testing and Validation:

Rigorously test new components in a staging environment before going live.
Ensure data consistency between batch and stream systems.

Continuous Improvement:

Gather feedback from users and stakeholders to identify further areas for optimization.
Continuously improve the architecture based on real-world usage patterns.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Summary: We will start by assessing the current architecture, setting up a PoC for streaming, and gradually integrating streaming into critical paths while maintaining batch processing for non-real-time needs. This approach ensures that we minimize disruptions and leverage existing assets effectively.

#### Common Mistakes

Ignoring the complexity of merging batch and stream processing.
Overlooking real-time data quality issues.
Failing to properly test new components before deployment.

#### Follow-Ups

How do you handle backfilling when introducing streaming?
What are the trade-offs between using open-source vs proprietary solutions for streaming?
How will you ensure data consistency across batch and stream layers?

**Code:**

```python
# Example of a simple Kafka consumer in Python using Flink's Python API
from pyflink.datastream import StreamExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = StreamExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

t_env.connect(FileSystem().path('input'))
     .with_format(OldCsv()
                  .field('name', DataTypes.STRING())
                  .field('value', DataTypes.DOUBLE()))
     .with_schema(Schema()
                  .field('name', DataTypes.STRING())
                  .field('value', DataTypes.DOUBLE()))
     .register_table_source('source')

t_env.connect(FileSystem().path('output'))
     .with_format(OldCsv()
                  .field('name', DataTypes.STRING())
                  .field('value', DataTypes.DOUBLE()))
     .with_schema(Schema()
                  .field('name', DataTypes.STRING())
                  .field('value', DataTypes.DOUBLE()))
     .register_table_sink('sink')

t_env.from_path('source') \
     .group_by(...) \
     .select(...) \
     .insert_into('sink')

t_env.execute("Stream Processing Job")
```

**Flow:**

1. Data Source
2. Kafka Broker
3. Apache Flink Streaming Job
4. Historical Data Warehouse
5. Real-Time Analytics Layer
6. Application Logic
7. Database/External Systems

