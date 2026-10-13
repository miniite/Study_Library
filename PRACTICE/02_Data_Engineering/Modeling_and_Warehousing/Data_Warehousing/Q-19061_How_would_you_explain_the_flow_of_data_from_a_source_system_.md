---
type: "practice"
difficulty: "Basic"
---

# How would you explain the flow of data from a source system into dashboards to a non-technical stakeholder?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Data Warehouse Fundamentals](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain the flow of data from a source system into dashboards to a non-technical stakeholder, we'll start with the basics and then delve into how different components interact in this process. Here’s a simple yet effective explanation:

#### Overview

Imagine you're trying to serve a delicious meal—your "source system" is like a garden where raw ingredients are grown, and your "dashboards" are like plates on which these ingredients are prepared and presented beautifully.

#### Triaging the Question

The core of this question involves understanding how data flows from an origin point (source) through various stages to reach its final destination (dashboard). The key components we need to explain include ETL (Extract, Transform, Load), data storage, and data visualization. We'll also touch on common challenges such as data quality and performance.

#### First Principles Solution

Source System: This is where your raw ingredients are sourced. It could be a database or a file store.

ETL Process:

Extract: Think of this step as picking the vegetables from the garden (extracting data).
Transform: Clean and prepare the vegetables before cooking them. In ETL, we clean, format, and modify the data to fit our needs.
Load: Finally, serve the prepared meal on a plate (load it into the warehouse or analytics database).

Data Storage: This is where all cleaned and transformed data is stored temporarily or permanently. Think of this as keeping your ingredients organized before cooking.

Processing Layer: Data might go through additional processing steps for analytics or real-time queries, ensuring it’s ready to be served.

Dashboards: The final stage where the prepared meal (analyzed data) is presented in a user-friendly format for decision-making.

#### Common Mistakes
and Follow-Ups:
Misunderstanding ETL Steps: Often, stakeholders confuse the steps or miss out on the importance of proper transformation.
Ignoring Data Quality Issues: Ensuring data accuracy and consistency is crucial but can be easily overlooked.
Performance Concerns: The system needs to handle large volumes of data efficiently.

#### Closing Summary

By explaining the flow from a source to dashboards using familiar analogies, we can make complex concepts more accessible. Highlighting these steps not only clarifies how the process works but also identifies potential areas for improvement and optimization.

**Comparison:**

```
Step	Action	Description
E	Extract	Fetch raw data from source system
T	Transform	Clean, format, and prepare the data
L	Load	Store processed data in a warehouse/database
```

**Code:**

```text
# Example of ETL Code (Python)
import pandas as pd

def extract(source_file):
    return pd.read_csv(source_file)

def transform(df):
    # Clean data, handle missing values
    df.dropna(inplace=True)
    return df

def load(transformed_data, target_db):
    transformed_data.to_sql('analytics_table', con=target_db.connect(), if_exists='replace')
```

**Example input / output:**

```text
Input: Extracted data from source SQL database with missing values.
Transformed Data:
- Cleaned up missing age for Jane (imputed to 25 based on common patterns).
Output: Loaded clean and transformed data into PostgreSQL analytics table.
```

**Data:**

```json
{
  "source_system": {
    "type": "SQL Database",
    "data": [
      {"id": 1, "name": "John", "age": 30},
      {"id": 2, "name": "Jane", "age": null}
    ]
  },
  "processed_data": [
    {"id": 1, "name": "John", "age": 30},
    {"id": 2, "name": "Jane", "age": 25} // Assumed age based on common patterns
  ],
  "target_database": {
    "type": "PostgreSQL",
    "table": "analytics_table"
  }
}
```

**Data:**

```
Cleaned Data
Source System
Extract
Transform
Load
Dashboard
```

