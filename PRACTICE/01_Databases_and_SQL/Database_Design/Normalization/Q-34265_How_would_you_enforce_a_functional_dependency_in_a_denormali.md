---
type: "practice"
difficulty: "Deep"
---

# How would you enforce a functional dependency in a denormalized warehouse table where the engine can't express constraints, using a data quality test?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Transitive Dependencies and 3NF](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Transitive_Dependencies_and_3NF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce a functional dependency in a denormalized warehouse table where the database engine cannot directly apply constraints, we need to rely on data quality tests and ETL processes. A functional dependency means that each value of one attribute (or set of attributes) uniquely determines another attribute. For instance, if customer_id functionally depends on first_name and last_name, then there should be no duplicate customer_id values for the same combination of names.

In a denormalized warehouse, such as an OLAP data mart, we often compromise normalization to gain performance benefits. However, this can lead to issues like duplicate keys if not managed properly. To ensure functional dependencies are maintained, we need to implement data quality tests and enforce these rules during ETL processes.

**Comparison:**

| customer_id | first_name | last_name |
|-------------|------------|-----------|
| 101         | John       | Doe       |
| 102         | Jane       | Smith     |
| 103         | John       | Doe       |   # Duplicate key issue
| 104         | Mike       | Wilson    |

**Code:**

```python
# Example Python pseudocode for a data quality test

def check_functional_dependency(data, key_columns, dependent_column):
    """
    Ensure that each value of the key columns uniquely determines the dependent column.

    :param data: DataFrame containing the table data
    :param key_columns: List of column names that should determine the dependent column
    :param dependent_column: The column name to which the keys must be unique
    :return: True if the dependency is enforced, False otherwise
    """
    # Group by key columns and check for duplicate dependent values
    grouped_data = data.groupby(key_columns)[dependent_column].nunique()
    return grouped_data.max() == 1

# Example usage
data = pd.DataFrame({
    'customer_id': [101, 102, 103, 104],
    'first_name': ['John', 'Jane', 'John', 'Mike'],
    'last_name': ['Doe', 'Smith', 'Doe', 'Wilson']
})

result = check_functional_dependency(data, key_columns=['first_name', 'last_name'], dependent_column='customer_id')
print("Functional dependency enforced:", result)
```

**Example input / output:**

```

data = pd.DataFrame({ 'customer_id': [101, 102, 103, 104], 'first_name': ['John', 'Jane', 'John', 'Mike'], 'last_name': ['Doe', 'Smith', 'Doe', 'Wilson'] })

result = check_functional_dependency(data, key_columns=['first_name', 'last_name'], dependent_column='customer_id') print("Functional dependency enforced:", result)

CODE
Copy
Output:

Functional dependency enforced: False
```

**Data:**

```json
{
  "data": {
    "customer_id": [101, 102, 103, 104],
    "first_name": ["John", "Jane", "John", "Mike"],
    "last_name": ["Doe", "Smith", "Doe", "Wilson"]
  }
}
```

**Flow:**

1. ETL Process Start
2. Check Functional Dependency
3. Enforce Rules
4. Load Data into WarehouseTable
5. Data Quality Test Post-Load
6. Notify If Violation Detected
7. Fix and Retry Load if Necessary

