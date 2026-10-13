---
type: "practice"
difficulty: "Intermediate"
---

# What is a surrogate key and why might you generate one during cleaning instead of trusting a messy natural key?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Cost of Data Cleaning](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Cost_of_Data_Cleaning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To start, let's quickly cover what a surrogate key is. A surrogate key is an artificial identifier that replaces the need for using natural keys (like customer names or order numbers) in data models. It ensures that data integrity and performance are maintained by providing a unique, immutable ID.

The core question here probes why we might want to use a surrogate key during cleaning instead of relying on messy natural keys. When dealing with real-world datasets, natural keys can be problematic due to their complexity, variability, or lack of uniqueness. During the cleaning process, it's common to encounter such issues that make natural keys unreliable.

#### Triaging the Question

The primary issue we're addressing is ensuring data integrity and consistency in a dataset. Natural keys often contain errors, duplicates, or inconsistencies that can compromise the quality of our analysis. By generating a surrogate key during cleaning, we ensure a robust, clean dataset without depending on potentially flawed natural keys.

#### Assumptions
The dataset contains messy natural keys.
We need to maintain data integrity and consistency.
Performance is a concern in large datasets.

#### First-Principles Solution

Let's break down the process step-by-step:

Identify Messy Natural Keys: These are the primary identifiers that we currently use, but they may have issues like missing values, typos, or duplicates.
Generate Surrogate Key: Create a unique identifier for each record in the dataset using an algorithm or manually.
Map Surrogate Key to Natural Key: Store both keys temporarily to ensure no data is lost during cleaning.
Clean Data Using Surrogate Key: Perform necessary cleanings (like removing duplicates, correcting errors) using the surrogate key.
Replace with Cleaned Data: Once cleaned, use the surrogate key to replace or update natural keys in the dataset.

#### Implementation
Example

Let's consider a simple example where we have customer data:

PLAINTEXT
CustomerID  | CustomerName  | Email               | Address
--------------------------------------------------
1           | John Doe      | jdoe@example.com    | New York, NY
2           | Jane Smith    | jane.smith          | Los Angeles, CA (misspelled)
3           | John Doe      | john.doe            | Chicago, IL
4           | Bob Johnson   | b.johnson@domain.com| Seattle, WA
Step 1: Identify Messy Natural Keys
CustomerName and Email have inconsistencies.
Address might also need cleaning.
Step 2: Generate Surrogate Key

We can use an algorithm like UUIDs to generate a unique identifier for each customer:

PLAINTEXT
SurrogateKey | CustomerID  | CustomerName  | Email               | Address
-----------------------------------------------------------------------
1            | 1           | John Doe      | jdoe@example.com    | New York, NY
2            | 2           | Jane Smith    | jane.smith          | Los Angeles, CA (misspelled)
3            | 3           | John Doe      | john.doe            | Chicago, IL
4            | 4           | Bob Johnson   | b.johnson@domain.com| Seattle, WA
Step 3: Map Surrogate Key to Natural Key

Store both keys temporarily:

PLAINTEXT
SurrogateKey | CustomerID  | CustomerName  | Email               | Address
-----------------------------------------------------------------------
1            | 1           | John Doe      | jdoe@example.com    | New York, NY
2            | 2           | Jane Smith    | jane.smith          | Los Angeles, CA (misspelled)
3            | 3           | John Doe      | john.doe            | Chicago, IL
4            | 4           | Bob Johnson   | b.johnson@domain.com| Seattle, WA

SurrogateKeyMap
SurrogateKey | CustomerID
----------------------
1            | 1
2            | 2
3            | 3
4            | 4
Step 4: Clean Data Using Surrogate Key

Perform cleanings:

Correct Email for Jane Smith.
Remove duplicate rows for John Doe.

After cleaning, the dataset looks like this:

PLAINTEXT
SurrogateKey | CustomerID  | CustomerName  | Email               | Address
-----------------------------------------------------------------------
1            | 1           | John Doe      | jdoe@example.com    | New York, NY
2            | 2           | Jane Smith    | jane.smith@domain.com| Los Angeles, CA (corrected)
4            | 4           | Bob Johnson   | b.johnson@domain.com| Seattle, WA
Step 5: Replace with Cleaned Data

Finally, use the surrogate key to replace or update natural keys:

PLAINTEXT
CustomerID  | CustomerName  | Email               | Address
--------------------------------------------------
1           | John Doe      | jdoe@example.com    | New York, NY
2           | Jane Smith    | jane.smith@domain.com| Los Angeles, CA (corrected)
4           | Bob Johnson   | b.johnson@domain.com| Seattle, WA

#### Closing Summary
and Common Mistakes

Using surrogate keys during cleaning ensures data integrity by providing a clean, unique identifier. This approach helps in handling messy natural keys more effectively and maintaining consistency throughout the dataset.

#### Common mistakes
include:

Not generating a unique key.
Failing to map the surrogate key back to the natural key.
Overlooking the need for robust error handling during mapping.

**Comparison:**

| SurrogateKey | CustomerID  | CustomerName  | Email               | Address                      |
|--------------|-------------|---------------|---------------------|------------------------------|
| 1            | 1           | John Doe      | jdoe@example.com    | New York, NY                  |
| 2            | 2           | Jane Smith    | jane.smith          | Los Angeles, CA (misspelled)   |
| 3            | 3           | John Doe      | john.doe            | Chicago, IL                   |
| 4            | 4           | Bob Johnson   | b.johnson@domain.com| Seattle, WA                  |
| SurrogateKeyMap         | CustomerID |
|-------------------------|------------|
| '6d3910c0-fa87-4e6c-a5e2-e62f51d06d66' | 1          |
| '8d3b2e05-c4a5-439e-bc3b-d08df7d4d20a' | 2          |
| '5c4d3c1f-8f6d-48f3-bec1-158701ddfc3a' | 3          |
| 'e8f6b94a-f0db-4394-a21d-c159a6c1d5f0' | 4          |
| Cleaned Data            | CustomerID  | SurrogateKey | Email               | Address                      |
|-------------------------|-------------|--------------|---------------------|------------------------------|
| 1                       | 1           | '6d3910c0-fa87-4e6c-a5e2-e62f51d06d66' | jdoe@example.com    | New York, NY                  |
| 2                       | 2           | '8d3b2e05-c4a5-439e-bc3b-d08df7d4d20a' | jane.smith@domain.com | Los Angeles, CA (corrected)   |
| 4                       | 4           | 'e8f6b94a-f0db-4394-a21d-c159a6c1d5f0' | b.johnson@domain.com | Seattle, WA                  |

**Code:**

```python
import uuid

# Example data
data = [
    {"CustomerID": 1, "CustomerName": "John Doe", "Email": "jdoe@example.com", "Address": "New York, NY"},
    {"CustomerID": 2, "CustomerName": "Jane Smith", "Email": "jane.smith", "Address": "Los Angeles, CA (misspelled)"},
    {"CustomerID": 3, "CustomerName": "John Doe", "Email": "john.doe", "Address": "Chicago, IL"},
    {"CustomerID": 4, "CustomerName": "Bob Johnson", "Email": "b.johnson@domain.com", "Address": "Seattle, WA"}
]

# Generate surrogate keys
surrogate_keys = [str(uuid.uuid4()) for _ in range(len(data))]

# Map surrogate keys to natural keys
surrogate_key_map = {sk: data[i]["CustomerID"] for i, sk in enumerate(surrogate_keys)}

for i, sk in enumerate(surrogate_keys):
    data[i]["SurrogateKey"] = sk

print("Original Data:")
print(data)
print("\nSurrogate Key Map:")
print(surrogate_key_map)

# Clean and update dataset
cleaned_data = [
    {"CustomerID": 1, "SurrogateKey": surrogate_keys[0], "Email": "jdoe@example.com", "Address": "New York, NY"},
    {"CustomerID": 2, "SurrogateKey": surrogate_keys[1], "Email": "jane.smith@domain.com", "Address": "Los Angeles, CA (corrected)"},
    {"CustomerID": 4, "SurrogateKey": surrogate_keys[3], "Email": "b.johnson@domain.com", "Address": "Seattle, WA"}
]

print("\nCleaned Data:")
print(cleaned_data)
```

**Example input / output:**

```text
Original Data:
[
    {'CustomerID': 1, 'CustomerName': 'John Doe', 'Email': 'jdoe@example.com', 'Address': 'New York, NY'},
    {'CustomerID': 2, 'CustomerName': 'Jane Smith', 'Email': 'jane.smith', 'Address': 'Los Angeles, CA (misspelled)'},
    {'CustomerID': 3, 'CustomerName': 'John Doe', 'Email': 'john.doe', 'Address': 'Chicago, IL'},
    {'CustomerID': 4, 'CustomerName': 'Bob Johnson', 'Email': 'b.johnson@domain.com', 'Address': 'Seattle, WA'}
]

Surrogate Key Map:
{
    '6d3910c0-fa87-4e6c-a5e2-e62f51d06d66': 1,
    '8d3b2e05-c4a5-439e-bc3b-d08df7d4d20a': 2,
    '5c4d3c1f-8f6d-48f3-bec1-158701ddfc3a': 3,
    'e8f6b94a-f0db-4394-a21d-c159a6c1d5f0': 4
}

Cleaned Data:
[
    {'CustomerID': 1, 'SurrogateKey': '6d3910c0-fa87-4e6c-a5e2-e62f51d06d66', 'Email': 'jdoe@example.com', 'Address': 'New York, NY'},
    {'CustomerID': 2, 'SurrogateKey': '8d3b2e05-c4a5-439e-bc3b-d08df7d4d20a', 'Email': 'jane.smith@domain.com', 'Address': 'Los Angeles, CA (corrected)'},
    {'CustomerID': 4, 'SurrogateKey': 'e8f6b94a-f0db-4394-a21d-c159a6c1d5f0', 'Email': 'b.johnson@domain.com', 'Address': 'Seattle, WA'}
]
```

**Data:**

```json
[
    {"CustomerID": 1, "CustomerName": "John Doe", "Email": "jdoe@example.com", "Address": "New York, NY"},
    {"CustomerID": 2, "CustomerName": "Jane Smith", "Email": "jane.smith", "Address": "Los Angeles, CA (misspelled)"},
    {"CustomerID": 3, "CustomerName": "John Doe", "Email": "john.doe", "Address": "Chicago, IL"},
    {"CustomerID": 4, "CustomerName": "Bob Johnson", "Email": "b.johnson@domain.com", "Address": "Seattle, WA"}
]
```

**Flow:**

1. Yes
2. No
3. Identify Messy Natural Keys
4. Generate Surrogate Key?
5. Map Surrogate Key to NaturalKey
6. Clean Data Using SurrogateKey
7. Replace with Cleaned Data
8. Proceed Without SurrogateKey
9. Final Cleaned Dataset

