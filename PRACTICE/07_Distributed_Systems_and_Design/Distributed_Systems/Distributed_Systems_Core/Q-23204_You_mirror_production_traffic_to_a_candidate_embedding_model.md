---
type: "practice"
difficulty: "Deep"
---

# You mirror production traffic to a candidate embedding model to compare retrieval quality without user impact. Explain how you'd diff the retrieved sets at scale, the sampling strategy that keeps it affordable, and how you avoid the shadow doubling your embedding cost.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Mixed Version Fleets](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively diff the retrieved sets from a candidate embedding model without impacting users, we need to implement a robust system that scales well and is both efficient and cost-effective. The core idea is to create a shadow database of user interactions, allowing us to compare the candidate model's performance against the production one without any disruptions.

The key challenge here involves diffing these large datasets efficiently while minimizing costs through smart sampling strategies. Here’s how we can approach this problem:

#### Overview
We will design a system that mirrors user interactions in real-time into a shadow database. This system uses a sophisticated sampling strategy to reduce the volume of data, making it feasible to compute differences between the candidate and production models.

#### Triage
The question probes our understanding of distributed systems, data processing pipelines, sampling techniques, and cost management strategies. We need to ensure that we can handle large-scale data efficiently while maintaining accuracy in performance metrics.

#### First-Principles Solution

Real-Time Mirroring: Set up a real-time pipeline using Apache Kafka or AWS Kinesis to capture user interactions from the production environment.
Shadow Database: Store these captured interactions in a shadow database such as Amazon DynamoDB or Google Bigtable, which are designed for high availability and scalability.
Sampling Strategy:
Random Sampling: Randomly sample a subset of data points to reduce the volume. This can be done using tools like Apache Spark.
Stratified Sampling: Ensure that samples are representative by stratifying users based on demographics or behavior patterns, reducing bias in the comparison.
Diff Computation: Implement an efficient diff computation algorithm that leverages distributed computing frameworks like Apache Flink or Spark. This will help us compare the candidate model’s performance against the production model at scale.
Cost Management:
Compute Resources: Optimize compute resources by running jobs during off-peak hours when costs are lower.
Data Redundancy: Avoid unnecessary data replication and ensure that only necessary data is stored in the shadow database.

#### Closing Summary
+ Common Mistakes/Follow-Ups: By implementing a robust system with careful sampling strategies, we can efficiently diff retrieved sets without significant cost increases or user disruptions. Key takeaways include the importance of representative samples and optimized resource usage to maintain both accuracy and efficiency.

#### Common mistakes
might involve over-sampling leading to high costs or under-sampling causing skewed results. Additionally, failing to consider off-peak hours for compute tasks could result in suboptimal performance due to increased costs.

**Comparison:**

```
Sample Method	Description
Random Sampling	Selects a subset of data points randomly. Can be biased but easier to implement.
Stratified Sampling	Divides the population into strata and samples from each stratum proportionally. Ensures representativeness.
```

**Code:**

```python
# Example of a basic sampling strategy using Apache Spark

from pyspark.sql import SparkSession
from pyspark.sql.functions import col, rand

spark = SparkSession.builder.appName("ShadowDBSampling").getOrCreate()

# Read production data into DataFrame
production_data = spark.read.parquet("/path/to/production/data")

# Randomly sample 1% of the data (adjust this percentage based on cost considerations)
sampled_data = production_data.sample(withReplacement=False, fraction=0.01)

# Save sampled data to shadow database
sampled_data.write.format("parquet").save("/path/to/shadow/database")
```

**Example input / output:**

```text
Input:
User interactions from production environment

Output:
Sampled and stored data in shadow database, ready for diff computation.
```

**Data:**

```json
{
  "user_interactions": [
    {"userId": "123", "itemId": "456", "timestamp": "2023-10-01T08:00:00Z"},
    {"userId": "789", "itemId": "101112", "timestamp": "2023-10-02T09:00:00Z"}
  ]
}
```

**Flow:**

1. Random Sampling
2. Stratified Sampling
3. Real-time User InteractionCapture
4. Sampling Strategy?
5. Store Sampled Data in ShadowDB
6. Ensure RepresentativenessBefore Storage
7. Distribute Diff ComputationJobs
8. Possibly Require More Stepsfor Preprocessing
9. Compute Differences BetweenModels

