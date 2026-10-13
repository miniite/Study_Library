---
type: "practice"
difficulty: "Intermediate"
---

# When would you still keep data in CSV or JSON instead of a columnar format?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Row vs Columnar Formats](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Row_vs_Columnar_Formats.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between using CSV or JSON over columnar formats like Parquet, ORC, or Avro, it's crucial to understand the trade-offs and specific use cases where traditional text-based formats might still be preferred.

CSV and JSON are text-based formats that offer simplicity and flexibility. They are widely supported by most programming languages and tools, making them accessible for quick prototyping and ad-hoc data processing. Columnar storage formats like Parquet and ORC optimize storage and query performance at the cost of additional complexity in handling and processing.

In summary:

Use CSV/JSON when simplicity, ease of integration, and flexibility are more critical.

#### Cons
ider columnar formats when performance and efficient querying are paramount.

Let's break this down step by step:

#### Triage
What is Being Probed

The interviewer is probing your understanding of the trade-offs between traditional text-based formats (CSV/JSON) and modern columnar storage formats. They want to see how you can apply first-principles reasoning to determine when simpler data structures are still appropriate.

#### Assumptions
and Context
Simplicity: CSV and JSON are easy to read, write, and process using common tools.
Performance Requirements: Columnar formats like Parquet and ORC optimize storage and query performance.
Tooling and Ecosystem Support: Most tools support text-based formats more widely.

#### First-Principles Solution

Simplicity and Flexibility:

CSV/JSON Usage: Use these when simplicity is key, such as in quick data exploration, ad-hoc scripts, or prototyping.

#### Example Scenario
1: A small dataset being used for exploratory data analysis (EDA) with Python's Pandas. CSV/JSON are straightforward to load and manipulate without significant overhead.

Performance Requirements:

Columnar Format Usage: Optimize performance-critical applications where query speed is essential, such as real-time analytics, business intelligence dashboards, or large-scale data processing.

#### Example Scenario
2: A high-frequency trading system requires ultra-fast query performance on a terabyte-sized dataset. Columnar formats like Parquet can significantly reduce scan times compared to scanning through CSV/JSON.

Tooling and Ecosystem Support:

CSV/JSON Ecosystem: Widely supported by various programming languages, libraries (Pandas in Python, Excel in R), and tools.
Columnar Format Tooling: More specialized but offer powerful features like distributed processing with Apache Spark.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake 1: Assuming columnar formats are always better due to performance. While they excel at specific use cases, simplicity and ease of use are often critical in development phases.

#### Follow-Up
Questions: The interviewer might ask about specific tools or scenarios where you would prefer CSV/JSON over columnar formats.

#### Example Scenario
s
PYTHON
# Example using Pandas for EDA with a small CSV file
import pandas as pd

data = pd.read_csv('small_dataset.csv')
print(data.head())
PYTHON
# Example using Spark to read Parquet for a large dataset
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("example").getOrCreate()
df = spark.read.parquet('large_dataset.parquet')
df.show(5)
Self-Check

This solution covers the key aspects of when CSV/JSON might still be preferred over columnar formats, providing concrete examples and addressing potential pitfalls.

**Code:**

```
Example using Pandas for EDA with a small CSV file

import pandas as pd

data = pd.read_csv('small_dataset.csv') print(data.head())

CODE
Copy

```python
# Example using Spark to read Parquet for a large dataset
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("example").getOrCreate()
df = spark.read.parquet('large_dataset.parquet')
df.show(5)
```

