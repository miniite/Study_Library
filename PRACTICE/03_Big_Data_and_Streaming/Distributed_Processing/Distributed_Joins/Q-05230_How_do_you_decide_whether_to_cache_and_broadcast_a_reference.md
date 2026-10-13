---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide whether to cache and broadcast a reference table or re-broadcast it per query?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reducing Shuffle for Large Joins](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Reducing_Shuffle_for_Large_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine whether to cache and broadcast a reference table or re-broadcast it per query in Apache Spark, we need to consider several factors including performance, memory usage, data size, and the nature of the queries. Broadcasting is particularly useful when dealing with small lookup tables that are referenced multiple times across different operations. Here's how you can approach this decision:

#### Triaging the Question

The question revolves around choosing between two methods: caching and broadcasting a reference table. Caching (using cache() or persist()) keeps the data in memory, while broadcasting (using broadcast()) sends the entire dataset to each executor for local processing.

Caching: Useful when the same data is used across multiple stages of an RDD/DataFrame transformation.
Broadcasting: Ideal for small tables that are frequently referenced but not modified during computations.

#### First-Principles Solution

First, let's understand the implications of each approach:

Caching (Persisting)

#### Pros
Reduces redundant data fetching and processing.
Can significantly speed up repeated operations on large datasets.

#### Cons

#### Cons
umes more memory, which can be a bottleneck for larger tables or when running multiple jobs concurrently.
Potential garbage collection issues if not managed properly.
Broadcasting

#### Pros
Reduces network overhead as the data is sent only once to each executor.
Ideal for small lookup tables that are frequently used in joins or filters.

#### Cons
Not suitable for large datasets, as it increases memory usage on executors.
May cause performance issues if not handled correctly.

To decide between these methods, consider the following factors:

Size of the Table: If the reference table is small (e.g., <10 MB), broadcasting is usually more efficient. For larger tables, caching might be a better option.
Frequency and Nature of Use: If the same data is used multiple times across different operations, caching can reduce redundant network traffic and improve performance.
Memory Constraints: Evaluate the available memory on your executors. If you're running several jobs concurrently or dealing with large datasets, broadcasting might consume too much memory.
Data Modification: Broadcasting a table means it cannot be modified during the job execution. Ensure that the data is static and will not change.
Practical Example

#### Cons
ider a scenario where we have a small lookup table small_table and a larger fact table fact_table. We want to join these tables multiple times in different transformations:

PYTHON
# Small lookup table
small_table = spark.createDataFrame([("A", 1), ("B", 2)], ["key", "value"]).cache()

# Large fact table
fact_table = spark.read.csv("large_fact_data.csv")

# Joining the tables using broadcasting for the small lookup table
result = fact_table.join(small_table.broadcast(), on="key")

In this example, we use broadcast() to cache and distribute small_table efficiently across executors.

#### Closing Summary
and Common Mistakes

To decide whether to cache or broadcast a reference table, consider the size of the dataset, its usage frequency, memory constraints, and data modification requirements. Broadcasting is generally more efficient for small lookup tables that are frequently used, while caching is better suited for larger datasets that need to be reused across multiple operations.

#### Common mistakes
include:

Using broadcast() for large datasets, leading to excessive memory usage.
Over-caching large datasets, causing performance degradation due to high memory consumption and frequent garbage collection.
Failing to consider the nature of data modification when using broadcasting.

**Comparison:**

        | Input DataFrame 1 | Input DataFrame 2 | Output Join Result |
        |------------------|------------------|--------------------|
        | key='A', value=1  | id=1, key='A'     | (id=1, key='A', value=1) |
        | key='B', value=2  | id=2, key='B'     | (id=2, key='B', value=2) |

**Code:**

```python
# Example code snippet
small_table = spark.createDataFrame([("A", 1), ("B", 2)], ["key", "value"]).cache()
fact_table = spark.read.csv("large_fact_data.csv")
result = fact_table.join(small_table.broadcast(), on="key")
```

**Example input / output:**

| Input DataFrame 1 | Input DataFrame 2 | Output Join Result |
|------------------|------------------|--------------------|
| key='A', value=1  | id=1, key='A'     | (id=1, key='A', value=1) |
| key='B', value=2  | id=2, key='B'     | (id=2, key='B', value=2) |

**Data:**

```plaintext
# Example data schema for small_table
key: string, value: integer
# Example data schema for fact_table (hypothetical)
id: integer, key: string, other_data: string
```

**Flow:**

1. Yes
2. No
3. Check Data Size
4. Small Table?
5. Use Broadcast()
6. Use Cache() / Persist()
7. Join Operation
8. Join Operation

