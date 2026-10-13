---
type: "practice"
difficulty: "Intermediate"
---

# How would you spot that a source table given to you is not in first normal form?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ First Normal Form](<../../../../01_Databases_and_SQL/Database_Design/Normalization/First_Normal_Form.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to determine if a provided table is in First Normal Form (1NF). A table is said to be in 1NF when it doesn't have any repeating groups or arrays within its columns. To spot that a source table isn’t in 1NF, we should follow these steps: identify and understand the schema of the given table, look for repeated column values across rows, and evaluate if the data can be split into separate tables.

To start, let's assume you receive a dataset with the following sample JSON format:

JSON
{
  "id": 1,
  "employee_name": "Alice",
  "address": [
    {"city": "New York", "state": "NY"},
    {"city": "Los Angeles", "state": "CA"}
  ],
  "department_id": [1, 2]
}

This JSON structure is a common representation of data that might not be in 1NF. Now, let’s dive into the first-principles solution.

**Comparison:**

```
Original Table Schema	Normalized Tables
id, employee_name, address[], department_id[]	person (id, name)
```

**Code:**

```python
# Sample input JSON
data = {
    "id": 1,
    "employee_name": "Alice",
    "address": [
        {"city": "New York", "state": "NY"},
        {"city": "Los Angeles", "state": "CA"}
    ],
    "department_id": [1, 2]
}
```

**Example input / output:**

```

To illustrate the problem, let's consider the above JSON structure. We need to check for repeating groups or arrays in columns:

Checking address: The address column is a list of dictionaries with repeated city-state pairs.
Checking department_id: The department_id column contains multiple values as an array.

These are clear indicators that the table is not in 1NF. Let's break this down step-by-step:

Step-by-Step Solution
1. Understand the Schema

First, we need to thoroughly understand the schema of the given data structure. In our example:

id: A unique identifier.
employee_name: The name of an employee.
address: A list of dictionaries representing addresses with city and state information.
department_id: A list of department IDs.
2. Identify Repeating Groups

Next, we check for any columns that contain arrays or lists where each value can be a separate entity. In our case:

The address column contains multiple address details, indicating it should be split into a separate table.
Similarly, the department_id column has multiple department IDs, suggesting it needs to be normalized.
3. Evaluate Normalization

To further validate this, we can transform the JSON structure into a normalized form and compare:

Normalized Form:

JSON
Copy
{
  "id": 1,
  "employee_name": "Alice"
}

{
  "address_id": 1,
  "city": "New York",
  "state": "NY",
  "employee_id": 1
}

{
  "address_id": 2,
  "city": "Los Angeles",
  "state": "CA",
  "employee_id": 1
}

{
  "department_id": 1,
  "employee_id": 1
}

{
  "department_id": 2,
  "employee_id": 1
}

In this normalized form, we have:

A person table for employee details.
An address table to store multiple addresses per person.
A department table and a junction table (e.g., employee_department) for department assignments.
Common Mistakes
Overlooking Simple Arrays: Sometimes the issue is straightforward, like in our example with address and department_id. However, more complex data structures can be easily missed.
Failing to Normalize Properly: After identifying 1NF issues, ensure that normalization follows a proper strategy (e.g., ensuring referential integrity).

By following these steps and using the provided examples, we can systematically identify when a given table is not in first normal form.
```

**Data:**

```json
{
  "id": 1,
  "employee_name": "Alice",
  "address": [
    {"city": "New York", "state": "NY"},
    {"city": "Los Angeles", "state": "CA"}
  ],
  "department_id": [1, 2]
}
```

**Flow:**

1. Yes
2. No
3. Initial Table
4. Check for Repeating Groups
5. Identify Non-1NF Columns
6. Split and Normalize Columns
7. Table is in 1NF
8. Transform to Normalized Form

