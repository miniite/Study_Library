---
type: "practice"
difficulty: "Basic"
---

# What does ETL stand for and what happens in each of the three stages?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ ETL vs ELT Design](<../../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/ETL_vs_ELT_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

ETL stands for Extract, Transform, Load. These are the three main steps involved in extracting raw data from various sources, transforming it into a consistent format, and then loading it into a target database or data warehouse. Let's break down each stage to understand what happens.

#### Triage
The question is asking us to define ETL and explain each of its stages. The interviewer might be probing your understanding of the fundamental concepts in data warehousing and ETL processes. We'll assume that the context here is a basic introduction, without delving into complex technical details or advanced topics.

**Code:**

```
Example code for extracting data from a source (simplified)

import pandas as pd

def extract_data(source): if source == 'CSV': return pd.read_csv('data.csv') elif source == 'API': import requests response = requests.get('http://api.example.com/data') return response.json()

CODE
Copy

```python
# Example code for transforming data (simplified)
def transform_data(data):
    # Cleaning and normalizing the data
    cleaned_data = data.dropna().astype(str).str.lower()
    # Adding derived columns
    cleaned_data['derived_column'] = cleaned_data['column_name'].apply(lambda x: x * 2)
    return cleaned_data
PYTHON
Copy
# Example code for loading data (simplified)
def load_data(data, target):
    if target == 'SQL':
        import sqlite3
        conn = sqlite3.connect('data.db')
        data.to_sql('table_name', con=conn, if_exists='replace', index=False)
        conn.close()
```

**Example input / output:**

```text
# Example input and output for the ETL process
source_data = extract_data('CSV')  # Extract step example
transformed_data = transform_data(source_data)  # Transform step example
load_data(transformed_data, 'SQL')  # Load step example
```

**Flow:**

1. Yes
2. No
3. Extract
4. Is data in structured format?
5. Parse and extract
6. Query API or database toretrieve raw data
7. C,D
8. Transform Data
9. Load into target storage

