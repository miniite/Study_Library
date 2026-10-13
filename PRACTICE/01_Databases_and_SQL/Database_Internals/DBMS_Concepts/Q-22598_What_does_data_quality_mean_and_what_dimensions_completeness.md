---
type: "practice"
difficulty: "Basic"
---

# What does data quality mean and what dimensions (completeness accuracy consistency timeliness uniqueness) do you check?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Late Arriving Dimensions](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Data quality is a fundamental aspect of any database management system (DBMS), ensuring that the data stored within it is accurate, complete, consistent, timely, and unique. As a fresher Data Engineer, understanding these dimensions is crucial to building robust data pipelines and databases.

To break down the question:

Dimensions of Data Quality: The interviewer is probing your knowledge of the core concepts that define data quality.
Key Assumptions: You need to explain each dimension in detail and discuss how they impact data integrity and the overall effectiveness of a database system.

Let's delve into each dimension step by step:

1. Completeness

Completeness refers to whether all required data elements are present in a dataset. For example, if you have a table for customer information, completeness would ensure that every record has fields such as name, address, and phone number filled out.

First Principle: Check each field for null values or missing entries.

#### Example Scenario
In a customers table with 100 records, identify which records lack an email address. This helps in understanding the gaps in data collection processes.
2. Accuracy

Accuracy ensures that the data is correct and free from errors. For instance, if you have a field for age, it should contain valid numerical values rather than strings or non-integer numbers.

First Principle: Validate each piece of data against predefined rules or business logic.

#### Example Scenario
In a sales transaction table, ensure that the product ID exists in the products table to avoid incorrect transactions.
3. Consistency

#### Cons
istency ensures that the same data elements are represented identically across multiple records and tables. For example, if you have customer names, they should be consistent (e.g., "John Doe" vs. "Johndoe").

First Principle: Use unique constraints or normalization techniques to enforce consistency.

#### Example Scenario
Ensure that the same customer ID is used throughout all related tables.
4. Timeliness

Timeliness refers to how up-to-date and relevant the data is. In a rapidly changing environment, ensuring that your database reflects current information is critical.

First Principle: Implement regular updates or refreshes of the data.

#### Example Scenario
For an inventory system, update stock levels based on real-time sales transactions.
5. Uniqueness

Uniqueness ensures that no duplicate records exist in a dataset. This prevents issues such as double billing or incorrect reports.

First Principle: Use primary keys and unique constraints to enforce this.

#### Example Scenario
In an orders table, ensure each order ID is unique across all records.

#### Closing Summary

In summary, data quality dimensions like completeness, accuracy, consistency, timeliness, and uniqueness are essential for maintaining the integrity and reliability of a database. As a Data Engineer, understanding these concepts helps in designing robust systems that can handle complex business needs and provide accurate insights.

#### Common Mistakes/Follow-Ups

Mistake: Focusing only on accuracy while neglecting completeness or timeliness.

#### Follow-Up
Discuss the importance of integrating automated data validation tools and regular quality checks into your database management practices.

**Comparison:**

| Customer ID | Name    | Email                    | Address        |
|-------------|---------|--------------------------|----------------|
| 1           | John Doe|                         | 123 Main St    |
| 2           | Jane Smith   | jane@example.com         |               |

**Code:**

```python
# Example Python code to check for missing values in a DataFrame
import pandas as pd

def check_completeness(data):
    # Check if any column has null values
    null_counts = data.isnull().sum()
    print("Null counts:", null_counts)
```

**Example input / output:**

```text
**Input:** Customers table with incomplete data.

- **Output:** A report indicating that the first customer is missing an email address, and the second customer has a missing address.
```

**Data:**

```json
{
  "customers": [
    {"name": "John Doe", "email": "", "address": "123 Main St"},
    {"name": "Jane Smith", "email": "jane@example.com", "address": ""}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Validated
4. Start
5. Is Data Missing?
6. Report Null Values
7. Validate Each Field AgainstRules
8. End
9. Fix and Revalidate

