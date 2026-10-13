---
type: "practice"
difficulty: "Basic"
---

# What is PII in a dataset and how would a data engineer mask or hash sensitive columns before sharing a table?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Composite vs Surrogate Keys](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Composite_vs_Surrogate_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

PII stands for Personally Identifiable Information, which refers to any piece of information that can be used on its own or with other information to identify an individual. In the context of data engineering, PII columns in a dataset are those that contain sensitive personal data such as names, addresses, social security numbers, etc., and they need to be protected when sharing datasets.

A data engineer's role is crucial in ensuring these sensitive columns are properly masked or hashed before sharing tables with external parties. This process not only protects the privacy of individuals but also complies with legal and regulatory requirements like GDPR, HIPAA, etc. The core idea behind masking or hashing PII is to make it unreadable while still retaining its integrity for certain operations.

#### Triaging the Question

The question asks about two main aspects:

Definition of PII: Understanding what personally identifiable information (PII) means in a dataset.
Methods to Mask or Hash Sensitive Columns: Techniques and considerations for protecting sensitive data before sharing tables.

#### Assumptions
We are dealing with a structured database table.
The goal is to ensure the data remains useful while preserving privacy.
Legal compliance and ethical standards must be considered.

#### Solution
From First Principles

Identifying PII Columns:

Start by identifying columns that could potentially contain sensitive personal information. Common types include names, addresses, phone numbers, email addresses, social security numbers, etc.

Masking Techniques:

Anonymization: Replace direct identifiers with generic values (e.g., replacing a name with "User-123").
Generalization: Aggregate or generalize the data to make it less specific (e.g., replacing an address with a city or state).
Tokenization/Hashing: Use hash functions or token generation to replace sensitive values while maintaining their integrity for certain operations.

#### Implementation
Steps:

Schema Inspection: Analyze the database schema to identify PII columns.
Data Masking Library: Utilize libraries like Apache Redact, Dapper, or custom scripts to apply masking functions.
Test and Validate: Ensure that the masked data still passes necessary checks (e.g., integrity constraints) and is compliant with regulations.

#### Example
Workflow

Let's consider a table users with columns: id, name, email, address.

Step 1: Identify PII Columns:

name: Possible to mask.
email: Mask or generalize.
address: Tokenize or anonymize.

Step 2: Apply Masking Functions:

PYTHON
import redact

def mask_user_data(row):
    row['name'] = "User-" + str(hash(row['id']) % 100)  # Example of tokenization
    row['email'] = "<EMAIL>"  # Generalized email
    return row

masked_rows = [mask_user_data(row) for row in original_rows]

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: Masking PII involves identifying sensitive columns, applying appropriate techniques like tokenization or generalization, and ensuring the data remains useful while complying with regulations.

#### Common Mistakes
Over-masking, which can render the dataset unusable.
Under-masking, leaving PII unprotected.
Failing to consider legal compliance.

#### Follow-Ups
What are the trade-offs between different masking techniques?
How do you ensure data integrity after applying these transformations?
Can you discuss an example scenario where hashing might not be suitable?

**Comparison:**

| ID | Name       | Email           | Address          |
|----|------------|-----------------|------------------|
| 1  | User-42    | <EMAIL> | 123 Main St      |
| 2  | User-56    | <EMAIL>| 456 Oak Ave      |

**Code:**

```python
import redact

def mask_user_data(row):
    row['name'] = "User-" + str(hash(row['id']) % 100)
    row['email'] = "<EMAIL>"
    return row

masked_rows = [mask_user_data(row) for row in original_rows]
```

**Example input / output:**

```text
original_rows = [
    {"id": 1, "name": "Alice", "email": "<EMAIL>", "address": "123 Main St"},
    {"id": 2, "email": "<EMAIL>", "address": "456 Oak Ave"}
]

masked_rows = [mask_user_data(row) for row in original_rows]
print(masked_rows)
```

**Data:**

```json
{
  "users": [
    {"id": 1, "name": "Alice", "email": "<EMAIL>", "address": "123 Main St"},
    {"id": 2, "name": "Bob", "email": "<EMAIL>", "address": "456 Oak Ave"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is PII Column?
5. Mask/Hash Data
6. Skip
7. Validate Masked Data
8. Export Table
9. End

