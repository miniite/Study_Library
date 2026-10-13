---
type: "practice"
difficulty: "Intermediate"
---

# What is the benefit of partitioning for cloud query costs when you pay per data scanned (like Athena)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Partitioning and Bucketing Design](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Partitioning_and_Bucketing_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Partitioning in databases, especially in the context of cloud services like Amazon Athena, helps manage and optimize data access by reducing the amount of data that needs to be scanned during queries. This can significantly lower query costs when you pay per data scanned. Here’s how it works:

TL;DR

By partitioning your data, you can filter out large portions of irrelevant data before sending a request to the cloud service, thereby minimizing the volume of data Athena scans and thus reducing the cost.

#### Triaging the Question

The question is asking about the benefits of using partitioning for optimizing query costs in services like Amazon Athena. The key point is understanding how partitioning affects the amount of data scanned during queries, which directly impacts the associated costs. Assume that you're dealing with a large dataset and want to optimize your queries without compromising performance.

#### First-Principles Solution
Understanding Data Storage: Imagine storing data in a directory structure where each folder represents a subset of records. For example, if you have sales data by year, you might create separate folders for 2020, 2021, etc.
Partitioning Strategy: When designing your table schema, partition the data based on one or more columns that frequently filter queries. In our example, this would mean storing and querying data by year first, then possibly month or another dimension.
Query Optimization with Partitions:
Athena uses the directory structure to determine which partitions (i.e., folders) to scan. If a query filters on year = 2020, only the 2020 folder needs to be scanned, reducing the amount of data Athena has to process.
This targeted scanning is particularly useful when combined with another concept called bucketing.
Bucketing

Bucketing further refines how data is stored and queried. For instance, within a year partition (e.g., 2020), you might create subdirectories for each month (jan, feb, etc.). This additional layer of organization allows Athena to quickly identify the exact files needed without scanning the entire directory.

#### Example Scenario

#### Cons
ider a scenario where you have sales data from 2018 to 2023, stored in an S3 bucket with a partitioning scheme based on year and month. If a user runs a query filtering for 2023-06, only the files corresponding to that specific time period would be scanned.

#### Closing Summary

By leveraging partitioning and bucketing, you can significantly reduce the amount of data Athena needs to scan for each query, thereby lowering query costs. This is particularly beneficial in environments where you are charged per byte of data scanned. For instance, if a year has 10 million records, scanning just one month (say 1% of the total) saves a substantial portion of the overall cost.

#### Common Mistakes
and Follow-Ups
Common Mistake: Not understanding that partitioning only affects which folders Athena needs to scan but not the total data size. If you have a large number of partitions, managing them can become complex.

#### Follow-Up
Discussing best practices for choosing the right columns to partition on and how to balance between query performance and cost savings.

**Comparison:**

```
Year	Month
2018	jan
2019	feb
2020	mar
2021	apr
2022	may
2023	jun
```

**Code:**

```sql
-- Example SQL to create a partitioned table in Amazon Athena
CREATE EXTERNAL TABLE sales_data (
    product_id INT,
    sale_date DATE,
    amount DECIMAL
)
PARTITIONED BY (year INT, month INT)
LOCATION 's3://my-bucket/sales-data/';
```

**Example input / output:**

```text
Input Query: SELECT * FROM sales_data WHERE year = 2023 AND month = 'jun';
Output: Scans files under s3://my-bucket/sales-data/2023/jun/
```

**Data:**

```json
{
  "year": [2018, 2019, 2020, 2021, 2022, 2023],
  "month": ["jan", "feb", "mar", "apr", "may", "jun", "jul", "aug", "sep", "oct", "nov", "dec"]
}
```

**Flow:**

1. Query Run
2. Filter by Year and Month
3. Scan only relevant partitions
4. Cost Reduction

