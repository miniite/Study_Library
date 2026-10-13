---
type: "practice"
difficulty: "Intermediate"
---

# What is a star schema, and how would a Python pipeline populate a fact table and its dimension tables in the right order?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling Schema Evolution in Python](<../../../../TOOLS/Python/Python_Language/Handling_Schema_Evolution_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

A star schema is a database design that optimizes data warehouse performance by separating facts from dimensions. In this context, we'll discuss how to populate a fact table and its dimension tables in Python using ETL (Extract, Transform, Load) processes.

#### Triaging the Question

The question revolves around understanding what a star schema is and implementing an ETL pipeline in Python to fill it correctly. The key points being probed are:

Understanding of star schema architecture.
Knowledge of how dimension tables relate to the fact table.
Ability to implement ETL processes, particularly focusing on loading data in the correct sequence.

#### Assumptions
and First Principles
Assume a basic understanding of SQL for database operations.
The fact table contains measures (numerical values) related to business events.
Dimension tables provide context about these measures through attributes like time, location, product details, etc.
ETL process will include three main stages: Extract, Transform, and Load.
Step-by-Step Solution
1. Define the Star Schema

A star schema consists of one or more fact tables and several dimension tables that store descriptive data related to these facts.

Fact Table Example:

PLAINTEXT
sales_fact (sale_id, product_id, customer_id, time_id, quantity_sold)

Dimension Tables:

time_dim: stores temporal information like date, year, month.
product_dim: stores product-related details like name, category.
customer_dim: stores customer-related data such as name, address.
2. Extract Data

Extract the raw data from sources (like CSV files, databases) into a staging area.

PYTHON
import pandas as pd

# Example of loading raw sales data
def extract_sales_data(file_path):
    return pd.read_csv(file_path)

# Load customer and product details
def extract_dim_data(dim_file_paths):
    customer_df = pd.read_csv(dim_file_paths['customer'])
    time_df = pd.read_csv(dim_file_paths['time'])
    product_df = pd.read_csv(dim_file_paths['product'])

    return customer_df, time_df, product_df

# Extract raw and dimension data
raw_sales_data = extract_sales_data('sales.csv')
customer_dim, time_dim, product_dim = extract_dim_data({
    'customer': 'customers.csv',
    'time': 'times.csv',
    'product': 'products.csv'
})
3. Transform Data

Transform the extracted raw data to fit into the star schema structure.

PYTHON
def transform_data(raw_sales_data, customer_dim, time_dim, product_dim):
    # Merge raw sales data with dimension tables
    transformed_data = pd.merge(
        left=raw_sales_data,
        right=time_dim[['time_id', 'date']],
        on='time_id',
        how='left'
    )

    transformed_data = pd.merge(
        left=transformed_data,
        right=customer_dim[['customer_id', 'name']],
        on='customer_id',
        how='left'
    )

    transformed_data = pd.merge(
        left=transformed_data,
        right=product_dim[['product_id', 'name', 'category']],
        on='product_id',
        how='left'
    )

    # Drop unnecessary columns
    final_sales_data = transformed_data.drop(columns=['time_id', 'customer_id', 'product_id'])

    return final_sales_data

# Transform the data
final_sales_data = transform_data(raw_sales_data, customer_dim, time_dim, product_dim)
4. Load Data into Fact and Dimension Tables

Load the transformed data into their respective tables.

PYTHON
import sqlite3

def load_into_fact_table(connection, fact_data):
    cursor = connection.cursor()
    cursor.execute('''
        INSERT INTO sales_fact (sale_id, quantity_sold, date, name, category)
        VALUES (?, ?, ?, ?, ?)
    ''',
                   (
                       tuple(fact_data['sale_id'].tolist()),
                       tuple(fact_data['quantity_sold'].tolist()),
                       tuple(fact_data['date'].tolist()),
                       tuple(fact_data['name'].tolist()),
                       tuple(fact_data['category'].tolist())
                   )
               )

def load_into_dimension_tables(connection, dim_data):
    for table_name, df in dim_data.items():
        cursor = connection.cursor()
        for _, row in df.iterrows():
            insert_query = f"INSERT INTO {table_name} VALUES ({', '.join(['?' for _ in range(len(row))])})"
            cursor.execute(insert_query, tuple(row))

    # Commit changes
    connection.commit()

# Connect to SQLite database
connection = sqlite3.connect('sales_database.db')

# Load data into tables
load_into_fact_table(connection, final_sales_data)
load_into_dimension_tables(connection, {'customer_dim': customer_dim, 'time_dim': time_dim, 'product_dim': product_dim})

# Close the connection
connection.close()

#### Closing Summary
and Common Mistakes/Follow-Ups

The star schema approach ensures efficient querying by separating fact tables from dimension tables. The ETL process in Python is crucial for handling large volumes of data efficiently. Key challenges include ensuring referential integrity, optimizing performance, and managing data quality.

#### Common Mistakes

Merging data incorrectly without proper keys.
Overwriting existing data instead of appending.
Not validating input data before transformation.
Closing

This ETL process ensures that the star schema is populated correctly with relevant and well-structured data. It's important to handle each step carefully, especially during transformation and loading phases.

**Comparison:**

```
Sales Fact Table
Time Dim
Customer Dim
Product Dim
Date
Name
Name, Category
```

**Code:**

```python
# Example code blocks for ETL pipeline
```

**Example input / output:**

```text
Input:
sales.csv, customers.csv, times.csv, products.csv

Output:
sales_fact table populated with transformed data.
customer_dim, time_dim, product_dim tables populated with customer, time, and product details respectively.
```

**Data:**

```json
{
  "sales.csv": [
    ["sale_id", "product_id", "customer_id", "time_id", "quantity_sold"],
    [1, 201, 301, 401, 5]
  ],
  "customers.csv": [
    ["customer_id", "name"],
    [301, "John Doe"]
  ],
  "times.csv": [
    ["time_id", "date"],
    [401, "2023-06-15"]
  ],
  "products.csv": [
    ["product_id", "name", "category"],
    [201, "Gadget X", "Electronics"]
  ]
}
```

**Flow:**

1. (Start)
2. Extract raw and dimensiondata
3. Transform the data
4. Load into tables
5. End

