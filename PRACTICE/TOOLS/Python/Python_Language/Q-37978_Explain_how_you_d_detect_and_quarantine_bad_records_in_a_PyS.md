---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd detect and quarantine bad records in a PySpark batch so one malformed row doesn't crash the whole job, routing rejects to a separate error path.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Data Reconciliation Scripts](<../../../../TOOLS/Python/Python_Language/Data_Reconciliation_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle malformed records in a PySpark batch without crashing the entire job, we need to implement a robust error detection and handling mechanism. This involves defining what constitutes a "bad record," processing each record individually to catch any exceptions, and then routing these errors to a designated error path for further analysis or logging.

#### Triage

The question is essentially asking how to design a PySpark batch process that can handle malformed input data without failing due to a single erroneous row. This involves understanding the structure of the data, defining validation logic, implementing exception handling in Python, and setting up an error processing mechanism. We will assume the raw data comes from a CSV file with potential issues like missing fields or incorrect datatypes.

#### First-Principles Solution
Data Validation: Define what constitutes a "bad record" based on the schema of the expected data.
Row-Level Processing: Process each row independently to ensure that if one row fails, it does not affect others.
Exception Handling: Catch and log exceptions for malformed rows.
Error Path Routing: Route rejected records to a separate path for further inspection or logging.
Detailed Steps
Define the Schema: Understand the expected structure of each record (e.g., columns with specific datatypes).
Validation Logic: Write validation functions that check if each row meets the schema requirements.
Row-Level Processing:
PYTHON
from pyspark.sql import SparkSession
from pyspark.sql.functions import col

# Initialize PySpark session
spark = SparkSession.builder.appName("Bad Record Detection").getOrCreate()

# Load raw data
df = spark.read.csv("path/to/raw_data.csv", header=True, inferSchema=True)

def validate_record(row):
    """Validate each row against the schema."""
    try:
        if not isinstance(row[col('column1')], int):  # Example validation check
            return False
        if not isinstance(row[col('column2')], float):  # Another example
            return False
        # Add more checks as needed
        return True
    except Exception as e:
        print(f"Error validating record: {str(e)}")
        return False

def process_data(df):
    """Process each row and handle validation errors."""
    valid_rows = []
    invalid_rows = []

    for row in df.collect():
        if validate_record(row):
            valid_rows.append(row)
        else:
            invalid_rows.append(row)

    # Handle valid rows (process as needed)
    processed_valid_rows = process_valid_rows(valid_rows)

    # Route invalid rows to an error path
    handle_invalid_records(invalid_rows)

    return processed_valid_rows

def process_valid_rows(rows):
    """Process valid rows."""
    # Example processing logic
    for row in rows:
        print(f"Processing valid row: {row}")

def handle_invalid_records(rows):
    """Handle and log invalid records."""
    # Write to an error file or a separate DataFrame
    invalid_df = spark.createDataFrame(invalid_rows, schema=df.schema)
    invalid_df.write.csv("path/to/error_data.csv")

processed_valid_rows = process_data(df)
Error Path Routing: Ensure that any malformed rows are written to a separate CSV file or stored in a DataFrame for later inspection.

#### Closing Summary
and Common Mistakes
Common Mistake 1: Handling all records in a single batch without proper error isolation can lead to job crashes.
Common Mistake 2: Not defining clear validation logic can result in undetected malformed records.
Common Mistake 3: Failing to log or route errors properly means issues may go unnoticed and unaddressed.

By implementing these steps, you ensure that a single bad record does not bring down the entire job while still providing valuable insights into the problematic data.

**Code:**

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col

# Initialize PySpark session
spark = SparkSession.builder.appName("Bad Record Detection").getOrCreate()

# Load raw data
df = spark.read.csv("path/to/raw_data.csv", header=True, inferSchema=True)

def validate_record(row):
    """Validate each row against the schema."""
    try:
        if not isinstance(row[col('column1')], int):  # Example validation check
            return False
        if not isinstance(row[col('column2')], float):  # Another example
            return False
        # Add more checks as needed
        return True
    except Exception as e:
        print(f"Error validating record: {str(e)}")
        return False

def process_data(df):
    """Process each row and handle validation errors."""
    valid_rows = []
    invalid_rows = []

    for row in df.collect():
        if validate_record(row):
            valid_rows.append(row)
        else:
            invalid_rows.append(row)

    # Handle valid rows (process as needed)
    processed_valid_rows = process_valid_rows(valid_rows)

    # Route invalid rows to an error path
    handle_invalid_records(invalid_rows)

    return processed_valid_rows

def process_valid_rows(rows):
    """Process valid rows."""
    for row in rows:
        print(f"Processing valid row: {row}")

def handle_invalid_records(rows):
    """Handle and log invalid records."""
    # Write to an error file or a separate DataFrame
    invalid_df = spark.createDataFrame(invalid_rows, schema=df.schema)
    invalid_df.write.csv("path/to/error_data.csv")

processed_valid_rows = process_data(df)
```

**Example input / output:**

```text
No specific input/output examples are required for this solution, but the code above processes a DataFrame `df` and separates valid rows from invalid ones. The output is written to "path/to/error_data.csv."
```

**Data:**

```plaintext
# Sample raw data schema
column1: int
column2: float
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Row Valid?
5. Process Valid Row
6. Route Invalid Record to ErrorPath
7. End

