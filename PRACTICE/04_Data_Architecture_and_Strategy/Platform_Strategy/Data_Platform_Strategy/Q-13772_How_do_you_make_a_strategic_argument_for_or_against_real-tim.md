---
type: "practice"
difficulty: "Deep"
---

# How do you make a strategic argument for or against real-time when 90% of business questions are answered by daily data — where does streaming genuinely earn its complexity cost?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Open vs Proprietary Formats](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Open_vs_Proprietary_Formats.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary Overview

In today's fast-paced world, the decision to adopt a real-time data platform is often driven by the need to handle high-frequency events and ensure that decisions are made as soon as possible. However, in scenarios where 90% of business questions can be answered with daily aggregated data, the complexity and cost of a streaming architecture might not justify its implementation. This article will delve into the strategic considerations for real-time data platforms versus traditional batch processing. We'll evaluate the business context, technical trade-offs, and long-term costs to determine whether streaming genuinely adds value.

#### Triaging the Question

The question revolves around making an informed decision between a real-time and a batch processing strategy based on the specific requirements of the organization. The key aspects being probed include:

Business Context: Understanding what types of questions are frequently asked, their urgency, and the impact of delayed responses.
Technical Feasibility: Assessing whether streaming technologies can be effectively implemented given existing infrastructure and skill set.
Cost-Benefit Analysis: Evaluating the financial costs associated with maintaining a real-time system versus the potential benefits it might offer.
Data Latency Impact: Determining if the 10% of queries that require timely data are critical enough to warrant the added complexity.

#### First-Principles Solution
Step 1: Define the Problem

The primary challenge is determining whether a real-time system is necessary when most business questions can be answered with daily summaries. This involves identifying the key use cases and their requirements:

Use Case Identification: Break down common business queries to understand which ones are time-sensitive.
Response Time Requirements: Determine if there's an acceptable delay for decision-making processes.
Step 2: Evaluate Business Context

Assess the types of questions being asked:

Operational Queries: These often require real-time insights, such as monitoring system health or identifying anomalies.
Historical Analysis: Queries that focus on trends and historical patterns can typically be handled with daily aggregated data.
Step 3: Technical Feasibility

Evaluate the technical infrastructure required for streaming:

Data Ingestion: The ability to ingest large volumes of high-frequency data without loss or corruption.
Processing Frameworks: Whether existing frameworks (like Apache Kafka, Flink) can be effectively used in the organization.
Skill Set and Training: Assess whether the team has the necessary expertise to manage a streaming platform.
Step 4: Cost-Benefit Analysis

Calculate the total cost of ownership for both real-time and batch processing:

Infrastructure Costs: The additional hardware, software licenses, and maintenance costs associated with a streaming platform.
Operational Costs: Increased operational overhead due to more complex systems.
Financial Benefits: Quantify any potential savings or revenue generated from faster decision-making.
Step 5: Case Study

#### Cons
ider a hypothetical scenario where an e-commerce company faces the choice between real-time and batch processing:

Scenario Setup:

Current System: Daily batch processing of user activities to generate insights.
Real-Time Requirement: Real-time fraud detection system with a response time requirement of under 1 second.

Analysis:

Current Process: User activities are ingested daily, processed, and aggregated into actionable insights. This takes about 24 hours but provides sufficient historical context for decision-making.
Real-Time System: A real-time streaming platform would provide faster anomaly detection but requires significant investment in infrastructure, ongoing maintenance, and skilled personnel.

Decision: Given the low frequency of fraud incidents (10% of total queries), the cost of implementing a complex real-time system does not justify the benefits. The current batch processing approach provides reliable insights with minimal latency.

Step 6: Long-Term Strategy

Develop a strategic plan that balances immediate needs and long-term goals:

Incremental Approach: Start by piloting small, targeted use cases where real-time data is critical.
Hybrid Model: Combine batch and stream processing to leverage the strengths of both approaches.
Training and Support: Ensure the team has the necessary skills and support to manage a hybrid platform.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In conclusion, making an informed decision between real-time and batch processing involves a holistic evaluation of business needs, technical feasibility, and cost-benefit analysis. While streaming can offer significant advantages in certain scenarios, for 90% daily data-driven queries, the added complexity might not be justified.

#### Common Mistakes

Overestimating Business Needs: Assuming all queries require real-time responses when many can be handled with delayed data.
Underestimating Costs: Failing to account for long-term infrastructure and operational costs.
Ignoring Technical Feasibility: Overlooking the existing technical capabilities and skill sets.

#### Follow-Ups

Discuss specific use cases in more detail.
Explore alternative architectures (e.g., hybrid models).

#### Cons
ider edge cases such as data consistency and privacy.

**Comparison:**

```
Use Case	Response Time Requirement	Volume
Fraud Detection	<1 second	High-frequency
Daily Insights	24 hours	Low-volume, low-frequency
```

**Code:**

```python
# Example code to demonstrate a simple batch processing pipeline using Apache Spark

from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Batch Processing").getOrCreate()

# Read daily logs
logs_df = spark.read.csv("daily_logs.csv")

# Process and aggregate data
aggregated_df = logs_df.groupBy("user_id").agg({"event": "count"})

# Write aggregated results to disk
aggregated_df.write.parquet("aggregated_results")
```

**Example input / output:**

```text
Input:
- Daily user activity logs (high volume, high frequency)

Output:
- Aggregated daily insights for fraud detection

Dry-run Example:
User Activity Logs -> Data Ingestion -> Batch Processing -> Aggregated Insights -> Storage
```

**Data:**

```json
{
  "use_case": {
    "name": "Fraud Detection",
    "response_time": "<1 second",
    "volume": "High-frequency"
  },
  "current_system": {
    "type": "Batch Processing",
    "response_time": "24 hours",
    "frequency": "Daily"
  }
}
```

**Flow:**

1. No
2. Yes
3. Ingest Data
4. Is Real-Time Required?
5. Batch Processing
6. Real-time Streaming Platform
7. Generate Aggregated Insights
8. Process and Alert onAnomalies
9. Store Results
10. Monitor and Act in Real-Time

