---
type: "practice"
difficulty: "Deep"
---

# How would you model an event-stream source into a fact table, deciding grain when events can arrive out of order and be duplicated?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fact Table Grain](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_Table_Grain.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model an event stream source into a fact table while handling out-of-order events and potential duplicates, we need to carefully consider the data's granularity (grain), ensure robustness against anomalies, and design a schema that supports efficient processing and querying. We'll use a star or snowflake schema as our foundation, with specific strategies for managing out-of-order and duplicate events.

The core challenge is to maintain accurate and consistent fact table entries despite the variability in event timing and content. To achieve this, we need to:

Define the grain of our fact table.
Implement mechanisms to handle out-of-order events without corrupting the data model.
Ensure that duplicate records are handled appropriately.

**Comparison:**

```
Sales ID	Order Date	Customer ID	Product ID	Quantity	Amount
1	2023-09-01	42	101	5	75.00
3	2023-09-02	43	102	10	150.00
```

**Code:**

```sql
-- Example SQL for defining a fact table in a star schema

CREATE TABLE sales_fact (
    sales_id BIGINT PRIMARY KEY,
    order_date DATE NOT NULL,
    customer_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL
);

-- Example of an incremental load function to handle out-of-order events

CREATE OR REPLACE FUNCTION process_sales_events(
    event_data JSONB
)
RETURNS void AS $$
DECLARE
    event RECORD;
BEGIN
    FOR event IN SELECT * FROM jsonb_array_elements(event_data) LOOP
        -- Process each event individually, ensuring correct grain and handling duplicates
        INSERT INTO sales_fact (sales_id, order_date, customer_id, product_id, quantity, amount)
        VALUES (
            COALESCE(event->>'sales_id', nextval('sales_sales_id_seq')),
            TO_DATE(event->>'order_date', 'YYYY-MM-DD'),
            event->>'customer_id'::INT,
            event->>'product_id'::INT,
            event->>'quantity'::INT,
            (event->>'amount')::DECIMAL(10, 2)
        )
        ON CONFLICT (sales_id) DO NOTHING; -- Handle duplicates by ignoring them
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

**Example input / output:**

```text
-- Example input data

SELECT '[
  {"sales_id": "1", "order_date": "2023-09-01", "customer_id": 42, "product_id": 101, "quantity": 5, "amount": "75.00"},
  {"sales_id": "2", "order_date": "2023-09-01", "customer_id": 42, "product_id": 101, "quantity": 5, "amount": "75.00"},
  {"sales_id": "3", "order_date": "2023-09-02", "customer_id": 43, "product_id": 102, "quantity": 10, "amount": "150.00"}
]'::JSONB AS event_data;

-- Example output data

SELECT process_sales_events(event_data) FROM (VALUES ('[
  {"sales_id": "1", "order_date": "2023-09-01", "customer_id": 42, "product_id": 101, "quantity": 5, "amount": "75.00"},
  {"sales_id": "2", "order_date": "2023-09-01", "customer_id": 42, "product_id": 101, "quantity": 5, "amount": "75.00"},
  {"sales_id": "3", "order_date": "2023-09-02", "customer_id": 43, "product_id": 102, "quantity": 10, "amount": "150.00"}
]')::JSONB);

-- Expected output (duplicates ignored)

SELECT * FROM sales_fact;
```

**Data:**

```json
{
  "events": [
    {
      "sales_id": "1",
      "order_date": "2023-09-01",
      "customer_id": 42,
      "product_id": 101,
      "quantity": 5,
      "amount": "75.00"
    },
    {
      "sales_id": "2",
      "order_date": "2023-09-01",
      "customer_id": 42,
      "product_id": 101,
      "quantity": 5,
      "amount": "75.00"
    },
    {
      "sales_id": "3",
      "order_date": "2023-09-02",
      "customer_id": 43,
      "product_id": 102,
      "quantity": 10,
      "amount": "150.00"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Event Stream Source
4. Is Event Out-of-Order?
5. Mark as Duplicate, Ignore
6. Process and Insert into FactTable

