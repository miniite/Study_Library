---
type: "practice"
difficulty: "Intermediate"
---

# What is the purpose of a staging table in a data pipeline, and how does it help produce clean training data?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Validating Join Cardinality](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Validating_Join_Cardinality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A staging table serves as an intermediate storage location in a data pipeline where raw or semi-processed data is temporarily stored before being loaded into the final target tables. Its primary purpose is to ensure that the data used for training models is clean, consistent, and reliable.

The question asks about the role of a staging table in producing clean training data, which we will address by first triaging the key components involved and then detailing how these components work together from first principles.

**Code:**

```sql
-- Example SQL Schema for Staging Table
CREATE TABLE raw_data (
    id INT,
    customer_name VARCHAR(255),
    transaction_amount DECIMAL(10, 2),
    transaction_date DATE,
    payment_method ENUM('credit_card', 'debit_card', 'cash')
);

CREATE TABLE staging_table (
    id INT,
    cleaned_customer_name VARCHAR(255),
    cleaned_transaction_amount DECIMAL(10, 2),
    cleaned_transaction_date DATE,
    cleaned_payment_method ENUM('credit_card', 'debit_card', 'cash')
);
```

**Example input / output:**

```text
-- Raw Data Input
INSERT INTO raw_data (id, customer_name, transaction_amount, transaction_date, payment_method)
VALUES
(1, 'John Doe', 50.23, '2023-10-01', 'credit_card'),
(2, 'Jane Smith', -98.76, '2023-09-30', 'invalid_payment_method');

-- Staging Table Output
INSERT INTO staging_table (id, cleaned_customer_name, cleaned_transaction_amount, cleaned_transaction_date, cleaned_payment_method)
SELECT
    id,
    CASE WHEN TRIM(UPPER(customer_name)) NOT IN ('JOHN DOE', 'JANE SMITH') THEN NULL ELSE TRIM(UPPER(customer_name)) END AS cleaned_customer_name,
    COALESCE(transaction_amount, 0) AS cleaned_transaction_amount,
    transaction_date,
    CASE payment_method
        WHEN 'credit_card' OR 'debit_card' OR 'cash' THEN payment_method
        ELSE NULL
    END AS cleaned_payment_method
FROM raw_data;
```

**Data:**

```json
{
    "raw_data": [
        {"id": 1, "customer_name": "John Doe", "transaction_amount": 50.23, "transaction_date": "2023-10-01", "payment_method": "credit_card"},
        {"id": 2, "customer_name": "Jane Smith", "transaction_amount": -98.76, "transaction_date": "2023-09-30", "payment_method": "invalid_payment_method"}
    ]
}
```

