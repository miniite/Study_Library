---
type: "practice"
difficulty: "Intermediate"
---

# How would you compute a running total of daily revenue from a large events table using Spark?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Computing SLIs with Spark](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Computing_SLIs_with_Spark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a running total of daily revenue from a large events table using Apache Spark, we need to process the data in a way that allows us to aggregate the revenue by date as it streams into the system. This involves understanding the structure of the input data and leveraging Spark's DataFrame API for efficient processing.

#### Triage

The core task here is to calculate the cumulative sum (running total) of daily revenues. The question is essentially asking how we can implement a window function in Spark to achieve this. Key assumptions include:

We have an events table with columns event_date and revenue.
We want to compute the running total of revenue for each day.

#### First Principles Solution

First, let's define our problem more concretely. We need to process a stream of events (each event representing some transaction) and calculate the cumulative sum of revenues by date. This can be achieved using Spark’s DataFrame API along with window functions such as WindowSpec.

Here are the steps:

Read the input data into a Spark DataFrame.
Define a window specification that defines the partitioning (orderBy) and framing logic for the running total.
Use the sum function over the defined window to compute the cumulative sum of revenues.
Output the result.

#### Implementation
SCALA
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.functions._
import org.apache.spark.sql.expressions.Window

// Initialize Spark session
val spark = SparkSession.builder()
  .appName("Running Total Daily Revenue")
  .getOrCreate()

// Sample input data schema: event_date, revenue
val eventsDF = Seq(
  ("2023-10-01", 100.5),
  ("2023-10-01", 200.7),
  ("2023-10-02", 150.3),
  ("2023-10-02", 250.9),
  ("2023-10-03", 300.1)
).toDF("event_date", "revenue")

// Define a window specification
val dailyWindowSpec = Window.partitionBy($"event_date").orderBy($"event_date").rowsBetween(Window.unboundedPreceding, Window.currentRow)

// Compute the running total of revenue for each day
val runningTotalDF = eventsDF.withColumn("running_total_revenue", sum($"revenue").over(dailyWindowSpec))

// Show the result
runningTotalDF.show()
Explanation
Reading Data: We start by reading our input data into a Spark DataFrame.
Defining Window Specification: Using Window.partitionBy and orderBy, we define how to partition the data (by date) and order it within those partitions. The rowsBetween(Window.unboundedPreceding, Window.currentRow) ensures that the running total is calculated cumulatively over all previous rows.
Aggregation with Window Function: We use sum over the defined window specification to compute the cumulative sum of revenue for each day.

#### Closing Summary

By leveraging Spark’s DataFrame API and window functions, we can efficiently calculate a running total of daily revenues from a large events table. This approach ensures that our solution is scalable and performs well even with big datasets.

#### Common Mistakes
and Follow-Ups
Mistake: Forgetting to include the orderBy clause in the window specification.

#### Follow-Up
Discussing how this solution can be extended to handle more complex aggregations or different time granularities (e.g., hourly, weekly).
Mistake: Not considering partitioning strategies for large datasets to optimize performance.

