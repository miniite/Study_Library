---
type: "practice"
difficulty: "Deep"
---

# Describe how you would use MapReduce counters and custom metrics to build data-quality checks into a batch pipeline.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Hadoop and Batch Processing](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/Hadoop_and_Batch_Processing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To integrate data quality checks into a Hadoop-based batch pipeline, leveraging MapReduce counters and custom metrics is crucial. This approach allows us to monitor and validate the integrity of processed data at various stages of the pipeline. Essentially, we'll be creating a robust system that not only processes data but also ensures its accuracy and consistency.

Firstly, let's triage the question: The interviewer is probing your understanding of MapReduce counters, custom metrics, and their application in ensuring data quality within a batch processing framework. You need to demonstrate how these tools can be used to track various aspects of the pipeline’s performance, such as success rates, error counts, and specific data validation metrics.

#### Assumptions

The Hadoop environment is configured with MapReduce.
We are dealing with a typical ETL (Extract, Transform, Load) process where data quality checks need to be implemented at different stages.
Custom metrics will include fields like valid_records, invalid_records, and processing_time.
Step-by-Step Solution

Define the Metrics:

Identify key metrics that are essential for validating data integrity, such as:
valid_records: Number of records that pass validation checks.
invalid_records: Number of records that fail validation checks.
processing_time: Time taken to process each record or batch.

Configure MapReduce Counters:

Use the RecordReader and Reducer classes in your custom Mapper and Reducer functions to increment these counters as needed.
JAVA
public class DataQualityMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String line = value.toString();
        // Perform data validation and increment counters as needed
        if (isValid(line)) {
            word.set("valid_records");
            context.getCounter(word).increment(1);
        } else {
            word.set("invalid_records");
            context.getCounter(word).increment(1);
        }
    }

    private boolean isValid(String line) {
        // Custom validation logic
        return true;
    }
}

Aggregate Metrics in Reducer:

The reducer will aggregate the counters to ensure that all validation results are collected.
JAVA
public class DataQualityReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        context.write(key, new IntWritable(sum));
    }
}

Monitor and Alerting:

Use custom metrics to trigger alerts or take corrective actions if certain thresholds are breached.
JAVA
public class DataQualityAlerts {
    private final int validRecordThreshold = 95;
    private final int invalidRecordThreshold = 5;

    public void checkMetrics(Map<String, Integer> metrics) {
        if (metrics.get("valid_records") < validRecordThreshold * metrics.get("total_records")) {
            System.err.println("Warning: Low validation rate");
        }
        if (metrics.get("invalid_records") > invalidRecordThreshold * metrics.get("total_records")) {
            throw new RuntimeException("High error rate detected");
        }
    }
}

Implement Data Quality Checks:

Integrate these checks into the ETL process to ensure data quality at every step.
JAVA
public class DataQualityETL extends Configured implements Tool {
    @Override
    public int run(String[] args) throws Exception {
        Job job = new Job(getConf(), "Data Quality Check");
        job.setJarByClass(DataQualityETL.class);
        job.setMapperClass(DataQualityMapper.class);
        job.setReducerClass(DataQualityReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        // Set counters and metrics
        job.addCounter(new Counter("valid_records", 0));
        job.addCounter(new Counter("invalid_records", 0));
        job.addCounter(new Counter("processing_time", 0));

        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        DataQualityAlerts alerts = new DataQualityAlerts();
        int status = job.waitForCompletion(true) ? 0 : 1;
        if (status == 0 && !alerts.checkMetrics(getCounters())) {
            System.out.println("Data quality check passed");
        } else {
            System.err.println("Data quality check failed");
        }
        return status;
    }

    public static void main(String[] args) throws Exception {
        int exitCode = ToolRunner.run(new DataQualityETL(), args);
        System.exit(exitCode);
    }
}

#### Closing Summary

In summary, integrating MapReduce counters and custom metrics into a Hadoop batch pipeline ensures that data quality checks are performed systematically. By defining relevant metrics, configuring them in the Mapper and Reducer classes, and aggregating results, we can monitor and validate each step of the ETL process effectively. This approach also enables us to set up alerts for critical failures, thereby enhancing overall system reliability.

#### Common Mistakes/Follow-Ups
Failing to define clear metrics that cover all aspects of data validation.
Not properly handling edge cases where no records might be processed.
Ignoring the importance of timely and accurate aggregation of counter results.
Overlooking the need for robust alerting mechanisms in real-world deployments.

**Comparison:**

```
Metric Name	Description
valid_records	Number of records that pass validation
invalid_records	Number of records that fail validation
processing_time	Time taken to process each record
```

**Code:**

```java
public class DataQualityMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String line = value.toString();
        if (isValid(line)) {
            word.set("valid_records");
            context.getCounter(word).increment(1);
        } else {
            word.set("invalid_records");
            context.getCounter(word).increment(1);
        }
    }

    private boolean isValid(String line) {
        // Custom validation logic
        return true;
    }
}

public class DataQualityReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        context.write(key, new IntWritable(sum));
    }
}

public class DataQualityAlerts {
    private final int validRecordThreshold = 95;
    private final int invalidRecordThreshold = 5;

    public void checkMetrics(Map<String, Integer> metrics) {
        if (metrics.get("valid_records") < validRecordThreshold * metrics.get("total_records")) {
            System.err.println("Warning: Low validation rate");
        }
        if (metrics.get("invalid_records") > invalidRecordThreshold * metrics.get("total_records")) {
            throw new RuntimeException("High error rate detected");
        }
    }
}

public class DataQualityETL extends Configured implements Tool {
    @Override
    public int run(String[] args) throws Exception {
        Job job = new Job(getConf(), "Data Quality Check");
        job.setJarByClass(DataQualityETL.class);
        job.setMapperClass(DataQualityMapper.class);
        job.setReducerClass(DataQualityReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        // Set counters and metrics
        job.addCounter(new Counter("valid_records", 0));
        job.addCounter(new Counter("invalid_records", 0));
        job.addCounter(new Counter("processing_time", 0));

        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        DataQualityAlerts alerts = new DataQualityAlerts();
        int status = job.waitForCompletion(true) ? 0 : 1;
        if (status == 0 && !alerts.checkMetrics(getCounters())) {
            System.out.println("Data quality check passed");
        } else {
            System.err.println("Data quality check failed");
        }
        return status;
    }

    public static void main(String[] args) throws Exception {
        int exitCode = ToolRunner.run(new DataQualityETL(), args);
        System.exit(exitCode);
    }
}
```

**Example input / output:**

```

Input:

CODE
Copy
[CSV file with data]

Output:

PLAINTEXT
Copy
Data quality check passed

OR

PLAINTEXT
Copy
Warning: Low validation rate
High error rate detected
```

**Data:**

```json
{
  "valid_records": 100,
  "invalid_records": 5,
  "total_records": 105
}
```

**Flow:**

1. Valid
2. Invalid
3. Input Data
4. Map Phase
5. Validation Check
6. Increment valid_records
7. Increment invalid_records
8. D1,D2
9. Reduce Phase
10. Aggregate Metrics
11. Alerts and Validation Checks

