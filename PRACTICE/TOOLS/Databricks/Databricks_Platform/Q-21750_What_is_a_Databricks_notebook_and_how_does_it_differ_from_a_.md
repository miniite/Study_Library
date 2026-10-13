---
type: "practice"
difficulty: "Basic"
---

# What is a Databricks notebook and how does it differ from a normal Python script?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Databricks Ingestion](<../../../../TOOLS/Databricks/Databricks_Platform/Databricks_Ingestion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A Databricks notebook is an interactive environment for writing, running, and sharing code to process data using Apache Spark. It differs from a typical Python script in several key ways, including its collaborative features and integrated environment with other tools.

In essence, a Databricks notebook is more than just a container for code; it's designed to facilitate the entire data science workflow. Here’s how it works:

TL;DR: A Databricks notebook provides an interactive coding experience within a Jupyter-like interface, making it ideal for data analysis and exploration. It offers features like cell-based editing, runtime environments, and integration with other tools, setting it apart from static Python scripts.

#### Triage
The question is essentially asking to compare the functionality and use cases of Databricks notebooks against traditional Python scripts. We need to highlight key differences in terms of environment, collaboration, and workflow management.

#### First-principles solution

Environment and Integration: A Databricks notebook runs on a managed Apache Spark cluster provided by Databricks. This means you can leverage all the capabilities of Spark without worrying about setting up or managing your own infrastructure.
Collaboration Features: Notebooks support real-time collaboration, version control, and sharing via links with team members. You can add comments, annotations, and even publish notebooks for public consumption.
Interactive Coding: Cells in a notebook allow you to run code snippets individually, making it easier to test and iterate on your data processing logic. This interactivity is not typically available in static Python scripts.
Automated Testing: Databricks offers automated testing capabilities within the notebook environment, allowing for quick validation of transformations or models.

#### Closing summary
+ common mistakes/follow-ups:

While a Databricks notebook provides a rich set of features for data processing and analysis, it’s important to understand when using these features might be overkill compared to writing a traditional Python script.

#### Common mistakes
include treating notebooks as the sole solution for complex production workloads without considering other tools or environments that may better suit your needs.

**Code:**

```python
# Example Databricks notebook code snippet
spark = SparkSession.builder.appName("example").getOrCreate()
data = [("James", "Sales", 3000), ("Michael", "Sales", 4600),
        ("Robert", "Sales", 4100), ("Maria", "Finance", 3000)]
columns = ["Name", "Department", "Salary"]
df = spark.createDataFrame(data, columns)
display(df)  # Interactive display
```

**Flow:**

1. Databricks Notebook
2. Spark Cluster
3. Real-time Collaboration
4. Automated Testing
5. Data Processing Capabilities

