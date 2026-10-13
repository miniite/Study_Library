---
type: "practice"
difficulty: "Deep"
---

# What is a slowly changing fact and how does it differ from a slowly changing dimension?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ SCD Type 2 vs Mini-dimensions](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/SCD_Type_2_vs_Mini-dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Slowly Changing Facts (SCFs) are a concept in data warehousing used to handle changes over time for facts, which are numerical attributes that measure a business event or transaction. They differ from Slowly Changing Dimensions (SCDs), which manage changes over time for dimensions, i.e., descriptive attributes of entities like customers or products. Understanding SCFs and SCDs is crucial for maintaining the accuracy and integrity of historical data in data warehouses.

**Comparison:**

```
FLOWCHART · MERMAID
erDiagram
    CUSTOMER ||--o{ SALES_FACT : made sales to (date)
    PRODUCT ||--o{ SALES_FACT : sold product of quantity (date)
    SALES_FACT {
        string sale_id
        int customer_id
        int product_id
        int quantity
        date effective_date
        date expiration_date
    }
```

**Code:**

```

-- Example SQL to insert a new fact record with an incrementing surrogate key INSERT INTO sales_fact (sale_id, customer_id, product_id, quantity, sale_date) VALUES (1001, 2001, 3001, 5, '2023-10-01');

CODE
Copy
```sql
-- Example SQL to update a fact record with an effective date approach for SCFs
UPDATE sales_fact SET quantity = 6, effective_date = '2023-10-01' WHERE sale_id = 1001 AND sale_date = '2023-10-01';
```

**Example input / output:**

```

Input (Initial Sale Record):

JSON
Copy
{
  "sale_id": 1001,
  "customer_id": 2001,
  "product_id": 3001,
  "quantity": 5,
  "sale_date": "2023-10-01"
}

Output (Updated Sale Record):

JSON
Copy
{
  "sale_id": 1001,
  "customer_id": 2001,
  "product_id": 3001,
  "quantity": 6,
  "effective_date": "2023-10-01"
}
```

**Data:**

```json
{
  "sale": {
    "sale_id": 1001,
    "customer_id": 2001,
    "product_id": 3001,
    "quantity": 5,
    "sale_date": "2023-10-01"
  },
  "updated_sale": {
    "sale_id": 1001,
    "customer_id": 2001,
    "product_id": 3001,
    "quantity": 6,
    "effective_date": "2023-10-01"
  }
}
```

**Flow:**

1. No
2. Yes
3. Initial Sale Record
4. Update Required?
5. Retrieve Old Record
6. Create New Record withEffective Date
7. Insert New Record
8. Update Existing Record withExpiration Date

