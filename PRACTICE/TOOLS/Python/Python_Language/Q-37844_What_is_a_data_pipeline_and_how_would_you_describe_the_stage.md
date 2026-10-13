---
type: "practice"
difficulty: "Basic"
---

# What is a data pipeline, and how would you describe the stages (extract, transform, load) of a simple Python batch pipeline you have built or studied?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling Schema Evolution in Python](<../../../../TOOLS/Python/Python_Language/Handling_Schema_Evolution_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A data pipeline is a system that moves, processes, and delivers data from its source to a destination in a structured manner. At a basic level, it consists of three main stages: Extract (E), Transform (T), and Load (L). These stages are commonly referred to as the ETL process. Let’s dive into each stage and understand how you might implement them using Python.

In your role as a data engineer, if you were tasked with building a simple batch pipeline, here is what the entire process would look like:

Extract: This involves fetching data from one or more sources. The source could be a database, CSV file, JSON file, or even an API.
Transform: Once the data is extracted, it needs to be cleaned and transformed into a format that’s suitable for loading into the target system.
Load: Finally, the processed data is loaded into a destination storage system like a data warehouse or a database.

Now let's take a look at how you might implement this in Python with some examples:

TL;DR:
Extract: Fetch data from sources.
Transform: Clean and modify the extracted data.
Load: Store the transformed data in a target system.

#### Triage

The question is asking for an explanation of what a data pipeline is, specifically focusing on its stages. The interviewer might be looking to understand your understanding of ETL processes and how you would implement them using Python. Key assumptions include familiarity with common data sources and storage systems, as well as basic knowledge of Python.

#### First-Principles Solution
Extract Stage: You can use libraries like pandas or sqlalchemy to fetch data from databases, CSV files, or APIs.
Transform Stage: Use pandas for cleaning and transforming the data, which might involve handling missing values, filtering, aggregating, etc.
Load Stage: Save the transformed data using libraries such as pandas or by writing to a database.

Here is an example of how you could implement these stages:

Extract
PYTHON
import pandas as pd

def extract_data(source):
    if source == 'csv':
        # Load CSV file
        df = pd.read_csv('data.csv')
    elif source == 'database':
        # Connect to the database and fetch data using SQLAlchemy or similar
        from sqlalchemy import create_engine
        engine = create_engine('postgresql://user:password@localhost/dbname')
        query = "SELECT * FROM my_table"
        df = pd.read_sql_query(query, engine)
    return df
Transform
PYTHON
def transform_data(df):
    # Example transformations:
    # 1. Handling missing values
    df.fillna(0, inplace=True)

    # 2. Filtering rows based on certain conditions
    df = df[df['column_name'] > 10]

    # 3. Aggregating data
    grouped_df = df.groupby('category_column').agg({'numeric_column': 'mean'})

    return grouped_df
Load
PYTHON
def load_data(df, destination):
    if destination == 'csv':
        df.to_csv('output.csv', index=False)
    elif destination == 'database':
        from sqlalchemy import create_engine
        engine = create_engine('postgresql://user:password@localhost/dbname')
        df.to_sql(name='transformed_table', con=engine, if_exists='replace', index=False)

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In a real-world scenario, you might need to handle more complex data sources and transformations. Additionally, error handling is crucial in production systems to ensure that the pipeline runs smoothly even when issues arise.

#### Common mistakes
include not properly handling missing or inconsistent data, failing to test the pipeline thoroughly, and not logging errors effectively. Interviewers often follow up with questions about how you would optimize performance, handle large datasets, integrate this with other tools (like Airflow for scheduling), or manage versioning of your pipelines.

**Code:**

```python
import pandas as pd

def extract_data(source):
    if source == 'csv':
        # Load CSV file
        df = pd.read_csv('data.csv')
    elif source == 'database':
        # Connect to the database and fetch data using SQLAlchemy or similar
        from sqlalchemy import create_engine
        engine = create_engine('postgresql://user:password@localhost/dbname')
        query = "SELECT * FROM my_table"
        df = pd.read_sql_query(query, engine)
    return df

def transform_data(df):
    # Example transformations:
    # 1. Handling missing values
    df.fillna(0, inplace=True)

    # 2. Filtering rows based on certain conditions
    df = df[df['column_name'] > 10]

    # 3. Aggregating data
    grouped_df = df.groupby('category_column').agg({'numeric_column': 'mean'})

    return grouped_df

def load_data(df, destination):
    if destination == 'csv':
        df.to_csv('output.csv', index=False)
    elif destination == 'database':
        from sqlalchemy import create_engine
        engine = create_engine('postgresql://user:password@localhost/dbname')
        df.to_sql(name='transformed_table', con=engine, if_exists='replace', index=False)
```

