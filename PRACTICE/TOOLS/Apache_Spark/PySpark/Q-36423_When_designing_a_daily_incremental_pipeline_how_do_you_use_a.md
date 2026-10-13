---
type: "practice"
difficulty: "Intermediate"
---

# When designing a daily incremental pipeline how do you use a watermark/high-water-mark column in PySpark to only read new source rows?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Evolution in PySpark](<../../../../TOOLS/Apache_Spark/PySpark/Schema_Evolution_in_PySpark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an incremental pipeline using PySpark, we need to ensure that our processing only reads and processes new data from the source. A common approach is to use a watermark column in conjunction with event time semantics. The key idea here is to maintain a high-water mark of the latest processed timestamp, so that any records older than this mark are ignored.

In essence, you're asking how to configure PySpark jobs to only process new data since the last run. Let's break down the steps involved:

Understand the Question: The question is essentially about setting up a mechanism in PySpark to read and process new rows from a source table based on their timestamp. A watermark column allows us to define a grace period (e.g., 5 minutes) during which we will still consider data that might have been delayed.

#### Assumptions
We assume the source data is a Delta Table or Spark DataFrame with a timestamp column, and we're processing this in a streaming context (as opposed to batch). The watermark column represents the event time of each record.

Step-by-Step Solution:

Define the watermark duration.
Use trigger and window for processing only new data.
Ensure your schema includes the necessary timestamp column(s) and the watermark column.

Here’s a more detailed breakdown:

Step 1: Schema Definition

First, ensure that your source DataFrame has both a key (like an ID) and a timestamp (to indicate when each record was last updated). For example:

PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()
df = spark.read.csv("path/to/source/data.csv", header=True, inferSchema=True)
Step 2: Define Watermark Duration

You need to specify how long you expect the event time data to be delayed. For example:

PYTHON
watermark_duration = "5 minutes"
Step 3: Define Window Specification

Use window specifications to define the processing logic and watermark.

PYTHON
from pyspark.sql import functions as F

window_spec = windowSpec = Window.partitionBy("id").orderBy(F.col("timestamp"))
Step 4: Processing Logic with Watermark

Now, use the when condition in conjunction with the watermark duration to filter out old records and only process new ones.

PYTHON
from pyspark.sql import streaming

@streaming.foreachBatch
def process_batch(batch_df, batch_id):
    # Define a watermark column for event time processing
    batch_df = batch_df.withWatermark("timestamp", watermark_duration)

    # Filter to keep rows that have not been seen before or are newer than the last processed timestamp
    filtered_df = batch_df.filter(
        (F.col("id") < F.lag(F.col("id"), default=F.lit(0)).over(window_spec)) |
        ((F.col("timestamp") > F.lag(F.col("timestamp"), default=F.lit(0)).over(window_spec)))
    )

    # Further processing logic here
    result = filtered_df.groupBy("id").agg(F.max("value"))

    # Write the processed data to a sink (e.g., another table)
    result.write.format("delta").mode("append").save("/path/to/sink")

streaming.start()

#### Closing Summary
and Common Mistakes

This solution ensures that only new records are processed in each batch, thanks to the watermark column. The common mistakes often include:

Not properly defining the watermark duration.
Ignoring the need for window specification.
Failing to filter out old records effectively.

