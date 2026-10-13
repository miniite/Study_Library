---
type: "practice"
difficulty: "Basic"
---

# Explain the difference between narrow and wide dependencies with a concrete example, and why only wide ones incur a shuffle stage boundary.

**Difficulty:** Basic · **Levels:** SDE-1/2, SDE-2/3 · [↑ Spark Skew and Spill](<../../../../TOOLS/Apache_Spark/Spark_Optimization/Spark_Skew_and_Spill.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain the concepts of narrow and wide dependencies in Spark, let's first understand what these terms mean. In Apache Spark, dependencies are used to define how data is distributed between tasks during execution. Narrow dependencies ensure that each partition of a dataset can be processed independently by a single task, whereas wide dependencies require shuffling data across multiple partitions.

Narrow dependencies are more efficient because they allow parallel processing without the overhead of data shuffling. Wide dependencies, on the other hand, lead to shuffle stages, which involve significant data movement and serialization/deserialization processes.

Let's consider a concrete example to illustrate these differences.

**Comparison:**

```
Department	Total Salary
Sales	11700
Finance	3000
IT	2500
Marketing	7000
CEO	9000
Department	Total Salary	Average Salary
Sales	11700	3900
Finance	3000	3000
IT	2500	2500
Marketing	7000	7000
CEO	9000	9000
```

**Code:**

```python
# Example code in Python using PySpark

from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("DependencyExample").getOrCreate()

data = [("James", "Sales", 3000),
        ("Michael", "Sales", 4600),
        ("Robert", "Sales", 4100),
        ("Maria", "Finance", 3000),
        ("Thomas", "IT", 2500),
        ("Kurt", "Marketing", 7000),
        ("Albert", "CEO", 9000)]

columns = ["Name", "Department", "Salary"]
df = spark.createDataFrame(data, columns)
```

**Example input / output:**

```

To create a narrow dependency, we can group employees by department and calculate the total salary within each department.

PYTHON
Copy
from pyspark.sql.functions import sum as _sum

# Calculate total salary per department (narrow dependency)
total_salaries = df.groupBy("Department").agg(_sum("Salary"))

To create a wide dependency, let's consider an operation that requires data to be shuffled between different partitions. For example, calculating the average salary for each employee in their respective departments.

PYTHON
Copy
# Calculate average salary per department (wide dependency)
average_salaries = total_salaries.withColumn("Average Salary", _sum("Salary") / count("*"))

In this case, average_salaries will incur a shuffle stage boundary because it needs to aggregate data across all partitions first before calculating the average.
```

**Data:**

```json
[
    {"Name": "James", "Department": "Sales", "Salary": 3000},
    {"Name": "Michael", "Department": "Sales", "Salary": 4600},
    {"Name": "Robert", "Department": "Sales", "Salary": 4100},
    {"Name": "Maria", "Department": "Finance", "Salary": 3000},
    {"Name": "Thomas", "Department": "IT", "Salary": 2500},
    {"Name": "Kurt", "Department": "Marketing", "Salary": 7000},
    {"Name": "Albert", "Department": "CEO", "Salary": 9000}
]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Total Salaries
6. Department = 'Sales'?
7. 11700
8. Other Departments
9. Average Salary
10. Department = 'Sales'?
11. 3900
12. Other Departments

