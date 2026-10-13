---
type: "practice"
difficulty: "Deep"
---

# How would you make a customer dimension GDPR/DPDP-erasable when historical facts reference it and you cannot break aggregate history?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Special Dimension Types](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Special_Dimension_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of making a customer dimension GDPR/DPDP-erasable while ensuring that historical facts referencing it remain intact, we need to carefully design our data model and implement appropriate mechanisms. The key is to ensure that personal data can be anonymized or deleted from the dimension without breaking historical aggregate summaries. This involves using techniques like hashing for anonymization, creating a history of changes, and employing proper indexing and querying strategies.

**Comparison:**

```
Customer Dimension	Facts Table
id: <hashed_id>	customer_id: <hashed_id>
name: [Anonymized]	order_amount: 100.00
	date: '2023-01-01'
```

**Code:**

```python
# Example Python code snippet to illustrate anonymizing customer IDs
import hashlib

def hash_customer_id(customer_id):
    return hashlib.sha256(str(customer_id).encode()).hexdigest()

hashed_customer_id = hash_customer_id(1234)
print(hashed_customer_id)
```

**Example input / output:**

```

Given the initial data:

Initial Customer Dimension:

YAML
Copy
customer_dimension:
  - id: 1234
  - name: John Doe

Facts Table:

YAML
Copy
facts_table:
  - customer_id: 1234
    order_amount: 100.00
    date: '2023-01-01'

After erasing the personal data, the customer_dimension table would be updated to use a hashed ID:

Updated Customer Dimension:

YAML
Copy
customer_dimension:
  - id: <hashed_id>
  - name: [Anonymized]

The facts_table remains unchanged, but now references the anonymized customer ID.
```

**Data:**

```yaml
customer_dimension:
  - id: 1234
  - name: John Doe

facts_table:
  - customer_id: 1234
    order_amount: 100.00
    date: '2023-01-01'
```

**Flow:**

1. Yes
2. No
3. Data Modeling
4. Is the customer dimensionneeded in historical facts?
5. Use a hashed ID for thedimension
6. Update thecustomer_dimension table withanonymized data
7. Ensure all fact references areupdated to new IDs
8. Keep the original dimensionas-is
9. Query and retrieve data usingappropriate joins

