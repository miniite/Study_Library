---
type: "practice"
difficulty: "Deep"
---

# Explain what a data contract is at a practical level and what you'd put in one for a source team feeding your bronze layer. Who does what when the contract is broken?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Layered Pipelines Bronze Silver Gold](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A data contract is essentially an agreement between two parties about how data should be structured, validated, and transformed before it reaches its destination. In a data architecture context, particularly involving a bronze layer as part of a data lake or warehouse setup, the contract helps ensure that all source teams feed accurate and consistent data into your system. For a source team feeding the bronze layer, this means defining how their output should align with predefined standards.

When creating a data contract for a source team, we need to consider several aspects:

Data Fields: Define the fields (columns) required by the bronze layer.
Field Types and Validation Rules: Specify the expected types of each field and validation rules to ensure consistency.
Transformation Requirements: Detail any transformations that must be applied before data can be ingested.
Error Handling Mechanisms: Outline how errors should be handled if the contract is not followed.

To illustrate this, let's break down what would go into a practical data contract for a source team feeding the bronze layer:

Data Fields and Types: Define each field name, its type (e.g., string, integer), and any subtypes (e.g., date formats).
Validation Rules: Specify rules such as minimum/maximum values, required fields, and data format constraints.
Transformation Steps: Detail any necessary transformations like date parsing, string normalization, or cleaning processes.
Error Handling Strategy: Define how to handle invalid data, including retries, logging mechanisms, and fallbacks.

The contract should be clear and unambiguous to minimize misinterpretation and errors during the feeding process. When a contract is broken, it can lead to significant issues like data quality problems, incorrect analytics, or system failures.

**Comparison:**

```
Field	Type	Rules
customer_id	string	not_empty
transaction_date	date	match_format(ISO8601)
amount	float	positive
product_name	string	no_special_chars
Validation Rule	Field	Description
not_empty	customer_id	Ensure the field is not empty.
match_format(ISO8601)	transaction_date	The date must follow ISO 8601 format.
positive	amount	Amount must be a positive number.
no_special_chars	product_name	No special characters allowed in names.
```

**Code:**

```plaintext
# Example Data Contract

### Fields
- `customer_id` (string)
- `transaction_date` (date)
- `amount` (float)
- `product_name` (string)

### Validation Rules
- `customer_id` must not be empty.
- `transaction_date` should follow ISO 8601 format.
- `amount` must be a positive number.
- `product_name` cannot contain special characters.

### Transformation Steps
1. Convert all date strings to ISO 8601 format.
2. Remove any non-alphanumeric characters from product names.

### Error Handling Strategy
- Log errors and retry after 5 minutes if the issue persists.
- Notify the source team via email in case of critical failures.
```

**Example input / output:**

```text
### Input Example

| customer_id | transaction_date     | amount  | product_name      |
|-------------|---------------------|---------|------------------|
| 123         | 04/05/2023 10:00 PM  | -50.00  | ProductA          |

### Expected Output

| customer_id | transaction_date     | amount  | product_name      |
|-------------|---------------------|---------|------------------|
| 123         | 2023-04-05T22:00:00Z | -50.0   | Product A         |

### Error Handling

Log: "Invalid transaction amount, must be positive."
Email: "Failed to ingest data for customer 123 due to invalid amount value. Please check your input schema."
```

**Data:**

```json
{
  "fields": [
    {
      "name": "customer_id",
      "type": "string"
    },
    {
      "name": "transaction_date",
      "type": "date",
      "format": "ISO8601"
    },
    {
      "name": "amount",
      "type": "float",
      "rules": ["positive"]
    },
    {
      "name": "product_name",
      "type": "string",
      "rules": ["no_special_chars"]
    }
  ],
  "validation_rules": [
    {
      "field": "customer_id",
      "rule": "not_empty"
    },
    {
      "field": "transaction_date",
      "rule": "match_format",
      "format": "ISO8601"
    },
    {
      "field": "amount",
      "rule": "positive"
    },
    {
      "field": "product_name",
      "rule": "no_special_chars"
    }
  ],
  "transformations": [
    {
      "field": "transaction_date",
      "step": "parse_iso8601"
    },
    {
      "field": "product_name",
      "step": "remove_special_chars"
    }
  ],
  "error_handling": {
    "strategy": "log_and_retry",
    "interval": 5,
    "critical_actions": ["email_source_team"]
  }
}
```

**Flow:**

1. Source Team Data
2. Validate Fields
3. Log Error & Retry
4. Retry Failed Ingestion
5. Transform Data
6. Ingest into Bronze Layer
7. Notify Source Team if CriticalError

