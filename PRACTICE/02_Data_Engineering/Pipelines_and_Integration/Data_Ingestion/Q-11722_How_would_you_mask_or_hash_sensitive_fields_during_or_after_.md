---
type: "practice"
difficulty: "Deep"
---

# How would you mask or hash sensitive fields during or after ingestion?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Ingestion Credentials](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Ingestion_Credentials.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle sensitive data securely, we need to mask or hash certain fields either during or after data ingestion. In this context, I'll explain how to implement these practices using Airbyte and Fivetran as our tools of choice.

#### Triaging the Question

The core question revolves around ensuring that sensitive information is protected throughout its lifecycle. Specifically:

What are sensitive fields? Fields containing personal identification information (PII), financial data, or any other confidential information.
How do we mask these fields? By replacing parts of the actual value with placeholders while retaining the structure.
How do we hash these fields? By converting them into a fixed-length string of characters using cryptographic algorithms.

#### First Principles Solution

Identifying Sensitive Fields: We need to define which columns in our source data contain sensitive information. For example, email, credit_card_number, and social_security_number.

Configuring Airbyte or Fivetran:

Airbyte Configuration:
Use the Masking feature within Airbyte's configuration.
Define a custom mask function for each sensitive field.
Fivetran Configuration:
Utilize Fivetran’s built-in data transformation capabilities.
Implement custom SQL queries to hash or mask sensitive fields before they are ingested.

Masking vs Hashing:

Masking: A common method is to replace characters with placeholders like *. For example, hashing the first four and last four digits of a credit card number while keeping the middle part masked.
Hashing: Use cryptographic functions like SHA-256 or bcrypt. This converts data into an unreadable format that can be used for verification purposes but not easily reversible.

#### Implementation
Example:

Using Airbyte, we might configure a transformation step in the airflow task.
In Fivetran, custom SQL could look something like this:
SQL
SELECT
  id,
  email AS masked_email,
  credit_card_number AS masked_credit_card,
  ... other columns ...
FROM source_table;

#### Closing Summary
+ Common Mistakes

To summarize, the key is to define sensitive fields and then use either masking or hashing techniques based on the sensitivity of the data. Both Airbyte and Fivetran offer ways to implement these practices effectively. A common mistake is not defining which fields are sensitive, leading to potential breaches.

Common Follow-Ups
How do you ensure compliance with regulations like GDPR?
What about performance considerations when applying transformations during ingestion?
Are there any trade-offs between masking and hashing?

**Comparison:**

| Sensitive Field | Masking Function       |
|-----------------|-----------------------|
| email           | mask_email             |
| credit_card_number | hash_credit_card      |

**Code:**

```python
# Example custom Airbyte transformation in Python
def mask_email(email):
    return f"{email[0]}****{email[-4:]}"

# Example Fivetran SQL query with masking
SELECT
  id,
  email AS masked_email,
  credit_card_number AS masked_credit_card,
  ... other columns ...
FROM source_table;
```

**Example input / output:**

| Input | Output          |
|-------|-----------------|
| John.Doe@example.com | J****.Doe@exa*mple.c**m |
| 123456789012345678    | e2f61d2b0e... |

**Data:**

```json
{
  "sensitive_fields": [
    { "field_name": "email", "masking_function": "mask_email" },
    { "field_name": "credit_card_number", "masking_function": "hash_credit_card" }
  ]
}
```

**Flow:**

1. Data Source
2. Filter Sensitive Fields
3. Apply Masking
4. Apply Hashing
5. Masked Data
6. Hashed Data
7. Integrated into Target System

