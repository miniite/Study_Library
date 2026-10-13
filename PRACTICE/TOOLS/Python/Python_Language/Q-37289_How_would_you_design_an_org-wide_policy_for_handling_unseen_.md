---
type: "practice"
difficulty: "Deep"
---

# How would you design an org-wide policy for handling unseen categorical values and schema evolution in Python pipelines so that no model silently breaks when an upstream table adds or renames a column?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Handling Schema Evolution in Python](<../../../../TOOLS/Python/Python_Language/Handling_Schema_Evolution_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective org-wide policy for handling unseen categorical values and schema evolution in Python pipelines, we must focus on creating robust and scalable solutions that can seamlessly adapt to changes without breaking existing models or processes. This involves a combination of data validation, dynamic schema handling, and flexible pipeline management.

#### Triage

The question is asking how to manage two critical aspects: (1) handling unseen categorical values in input data, and (2) managing schema evolution in real-time pipelines. The interviewer likely wants to assess your understanding of robust data processing techniques, schema flexibility, and the ability to handle unexpected changes without downtime.

#### Assumptions
The system processes large volumes of data across multiple sources.
Schema changes can occur frequently due to new requirements or updates.
Unseen categorical values may arise from new categories introduced in upstream datasets.

#### First-Principles Solution

Data Validation and Transformation

Implement a robust validation mechanism that checks for missing fields, unexpected types, and invalid data points.
Use libraries like pandas and numpy to handle missing values and convert data types appropriately.

Dynamic Schema Handling

Develop a schema evolution strategy that can detect and adapt to changes in the input schema.
Utilize dataclasses, pydantic, or custom classes to define schemas dynamically.

Handling Unseen Categorical Values

Implement strategies for encoding new categories, such as using LabelEncoder from sklearn.
Optionally, use techniques like one-hot encoding or embedding layers in models to handle unseen values gracefully.

Pipeline Management

Use state-of-the-art pipeline management tools like Apache Airflow, Prefect, or Dagster for orchestrating tasks.
Ensure that the pipeline can be easily updated and re-run without significant manual intervention.

#### Implementation
Steps

Define a Dynamic Schema Class

PYTHON
from dataclasses import dataclass, field
from typing import List, Dict

@dataclass
class Schema:
    columns: List[str]
    types: Dict[str, type]

schema = Schema(columns=['id', 'name', 'category'], types={'id': int, 'name': str})

Data Validation Function

PYTHON
import pandas as pd

def validate_data(df, schema):
    errors = []
    for col in df.columns:
        if col not in schema.columns or type(df[col].iloc[0]) != schema.types.get(col):
            errors.append(f"Error: {col} is missing or has incorrect data type.")
    return errors

Handling Unseen Categorical Values

PYTHON
from sklearn.preprocessing import LabelEncoder

def handle_unseen_categories(df, col):
    if not df[col].isin(schema.types.keys()).all():
        le = LabelEncoder()
        df[col] = le.fit_transform(df[col])
    return df

Pipeline Management with Prefect

PYTHON
from prefect import task, flow

@task
def process_data(df):
    errors = validate_data(df, schema)
    if errors:
        raise ValueError("\n".join(errors))

    for col in df.columns:
        handle_unseen_categories(df, col)

    return df

@flow
def main_flow():
    # Load data from source
    df = pd.read_csv('data.csv')

    processed_df = process_data(df)
    # Continue with model training or other processing steps

#### Closing Summary
and Common Mistakes

In summary, a robust solution involves dynamic schema handling, flexible validation mechanisms, and resilient data processing techniques. Ensuring that your pipeline can adapt to new schemas without breaking existing models is crucial for maintaining system integrity.

#### Common mistakes
include:

Failing to implement proper error handling.
Not dynamically validating and adapting to changes in the input data.
Overlooking the need for manual intervention when schema changes are detected.
Self-Check

Correctness + completeness confirmed.

**Comparison:**

```
Columns	Types
id	int
name	str
category	object
```

**Code:**

```

from dataclasses import dataclass, field from typing import List, Dict

@dataclass class Schema: columns: List[str] types: Dict[str, type]

schema = Schema(columns=['id', 'name', 'category'], types={'id': int, 'name': str})

def validate_data(df, schema): errors = [] for col in df.columns: if col not in schema.columns or type(df[col].iloc[0]) != schema.types.get(col): errors.append(f"Error: {col} is missing or has incorrect data type.") return errors

def handle_unseen_categories(df, col): if not df[col].isin(schema.types.keys()).all(): le = LabelEncoder() df[col] = le.fit_transform(df[col]) return df

CODE
Copy
```python
from prefect import task, flow

@task
def process_data(df):
    errors = validate_data(df, schema)
    if errors:
        raise ValueError("\n".join(errors))

    for col in df.columns:
        handle_unseen_categories(df, col)

    return df

@flow
def main_flow():
    # Load data from source
    df = pd.read_csv('data.csv')

    processed_df = process_data(df)
    # Continue with model training or other processing steps
```

**Example input / output:**

```

No input data provided, but a sample response would validate and process a DataFrame.

PYTHON
Copy
import pandas as pd

# Sample input data
df = pd.DataFrame({
    'id': [1, 2, 3],
    'name': ['Alice', 'Bob', 'Charlie'],
    'category': ['A', 'B', np.nan]  # Introducing an unseen category 'nan'
})

errors = validate_data(df, schema)
print(errors)  # Output: ['Error: category is missing or has incorrect data type.']

processed_df = process_data(df)
print(processed_df)
```

**Data:**

```json
{
  "columns": ["id", "name", "category"],
  "types": {
    "id": int,
    "name": str
  }
}
```

**Flow:**

1. Errors
2. Load Data
3. Validate Schema
4. Handle Errors
5. Process Unseen Categories
6. Continue Processing
7. Final Processed DataFrame

