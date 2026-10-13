---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a generic test and a singular test in dbt?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ dbt Tests](<../../../../TOOLS/dbt/dbt_Core/dbt_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In dbt, both generic tests and singular tests serve to ensure data integrity, but they operate on different principles and are suited to different use cases.

TL;DR:
Generic Tests: These apply a pre-defined test logic across all rows of the model.
Singular Tests: These apply the same test logic to each individual row of the model. They require specifying an identifier column for each record.

#### Triage

The question is essentially asking about two types of tests in dbt: generic and singular, highlighting their differences and use cases. The interviewer is likely probing your understanding of dbt's testing mechanisms and how they can be effectively used to validate data integrity.

#### First-Principles Solution

Generic Tests:

Definition: These are pre-defined test rules that apply a common logic across all rows in the model.

#### Example
unique (ensures each row is unique), nonnegative (checks for non-negative values).
How They Work: Generic tests run on every record without needing to specify an identifier. For instance, if you want to ensure no duplicates exist in a column, the unique test will automatically apply this rule across all rows.

Singular Tests:

Definition: These are custom tests that can be defined by users and require specifying a unique identifier for each record.

#### Example
If you need to check if an order has been shipped after it was received, you would use a singular test with the order_id as the identifier.
How They Work: Singular tests are more flexible and allow for custom validation logic. For instance, if you have a model of orders, and want to ensure that each order is either in transit or delivered (and not both), you can define such a test.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Generic tests are straightforward and useful for common validations like ensuring data uniqueness or non-negativity. They require minimal configuration but lack flexibility since they cannot handle complex, per-record logic.

Singular tests, on the other hand, offer more control over validation rules by allowing you to define custom test conditions. However, they require specifying an identifier column, which can make your models slightly more verbose and may add complexity if not handled properly.

#### Common Mistakes
Overusing Generic Tests: While generic tests are convenient, relying too heavily on them might overlook specific business requirements that need singular testing.
Misunderstanding Identifier Columns: For singular tests, correctly identifying the columns used to uniquely identify each record is crucial. Incorrect identifiers can lead to false positives or negatives.

#### Follow-Ups
Can you give an example where both generic and singular tests would be used in a real-world scenario?
How do these testing mechanisms fit into a larger data validation strategy?

**Comparison:**

| Generic Tests          | Singular Tests                       |
|------------------------|--------------------------------------|
| Checks common logic    | Customizable per-record logic        |
| No need for identifier | Requires specifying unique columns   |

**Code:**

```python
# Example of using dbt's unique test (generic)
unique_test:
  config:
    unique_key: "customer_id"  # Assuming this is the unique key for customers

# Example of defining a custom singular test
singular_test:
  config:
    identifier_columns: ["order_id"]  # Custom logic to check order status
  sql: "
    select order_id,
           case when shipped_at < received_at then 'Invalid Status' else 'Valid Status' end as validation_result
    from orders"
```

**Example input / output:**

```
Generic Test Example:
SQL
Copy
select * from {{ ref('my_model') }} where _dbt_unique_key is not null and _dbt_unique_key = 'customer_id'
Singular Test Example:
SQL
Copy
select order_id,
       case when shipped_at < received_at then 'Invalid Status' else 'Valid Status' end as validation_result
from {{ ref('orders') }}
```

**Data:**

```yaml
# dbt.yml configuration snippet for testing models
models:
  my_project:
    my_model:
      tests:
        - unique: {}
        - singular_test:
            identifier_columns: ["order_id"]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Generic Test
6. Is the test applicable to allrows?
7. Run test across model
8. Singular Test
9. Do you have a uniqueidentifier for each row?
10. Define and apply singular test
11. Error: Singular tests requireunique identifiers

