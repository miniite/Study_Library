---
type: "practice"
difficulty: "Expert"
---

# How do you model a payments transaction fact so it supports both real-time fraud scoring and end-of-day reconciliation without two separate schemas?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Transaction vs Snapshot Facts](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Transaction_vs_Snapshot_Facts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model a payments transaction fact that supports both real-time fraud scoring and end-of-day reconciliation, we need to carefully design our database schema to accommodate both requirements while minimizing redundancy and ensuring data integrity. The key challenge is to balance between the needs of real-time processing, which requires quick access and updates, and batch processing for reconciliation, which can afford more overhead.

#### Triage

The question asks us to model a payments transaction fact in such a way that it supports two distinct but related use cases: real-time fraud scoring and end-of-day reconciliation. Real-time fraud scoring needs immediate and low-latency data access, whereas end-of-day reconciliation processes large volumes of transactions at the end of each day.

#### First-Principles Solution

To address both requirements, we can employ a hybrid approach combining elements of star and snowflake schemas. The central fact table will capture all transaction details, while dimensions will be used to support various analyses. We'll introduce a time-based partitioning strategy and a staging area for real-time data.

Central Fact Table: This will store the core transactional facts.
Time-Stamped Partitions: To ensure that real-time fraud scoring can access recent transactions efficiently.
Staging Area: For capturing real-time transactions before they are processed by end-of-day reconciliation.
Dimension Tables: Supporting both real-time and batch processing needs.
Step 1: Central Fact Table

The central fact table Transactions will store the core transactional data:

SQL
CREATE TABLE Transactions (
    transaction_id INT PRIMARY KEY,
    payment_id VARCHAR(50),
    amount DECIMAL(10,2),
    currency_code CHAR(3),
    status VARCHAR(20),
    fraud_score INT,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);
Step 2: Time-Stamped Partitions

To support real-time fraud scoring, we'll partition the Transactions table by date:

SQL
CREATE TABLE Transactions (
    transaction_id INT PRIMARY KEY,
    payment_id VARCHAR(50),
    amount DECIMAL(10,2),
    currency_code CHAR(3),
    status VARCHAR(20),
    fraud_score INT,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,

#### CONS
TRAINT chk_created_at CHECK (created_at >= DATEADD(day, -30, CURRENT_TIMESTAMP))
)
PARTITION BY RANGE (DATE(created_at));

This partitioning ensures that real-time queries can focus on recent transactions without scanning the entire table.

Step 3: Staging Area

We'll create a staging area for capturing and processing real-time data:

SQL
CREATE TABLE RealTimeTransactions (
    transaction_id INT,
    payment_id VARCHAR(50),
    amount DECIMAL(10,2),
    currency_code CHAR(3),
    status VARCHAR(20),
    created_at TIMESTAMP
);

Real-time fraud scoring will read from this staging area instead of the main Transactions table.

Step 4: Dimension Tables

Dimension tables support both real-time and batch processing:

PaymentMethods
Currencies
Statuses

These can be designed as star or snowflake schemas to accommodate various levels of granularity required by different analyses.

#### Closing Summary

By combining a central fact table with time-stamped partitions, a staging area for real-time data, and well-designed dimension tables, we can effectively model payments transactions that support both real-time fraud scoring and end-of-day reconciliation. This hybrid approach ensures efficient access and processing while maintaining data integrity and consistency.

#### Common Mistakes/Follow-Ups
Ignoring Real-Time Requirements: Failing to partition the main transaction table can lead to inefficient queries during real-time fraud scoring.
Overcomplicating Dimension Tables: Not designing dimension tables adequately for both use cases might result in redundant storage or insufficient granularity.
Not Using Staging Areas: Directly querying the main Transactions table for real-time operations without a staging area can lead to performance issues.

For follow-ups, we could discuss indexing strategies, data validation processes, and how to handle edge cases like transaction rollbacks during reconciliation.

**Comparison:**

```
Transaction Attributes	Description
transaction_id	Unique identifier for each transaction.
payment_id	Identifier linking to a specific payment method.
amount	Amount of the transaction in the specified currency.
currency_code	Three-letter ISO currency code.
status	Status of the transaction (e.g., completed, failed).
fraud_score	Score indicating potential fraudulent activity.
created_at	Timestamp when the transaction was created.
Partition Strategy	Description
RANGE (DATE(created_at))	Partitions transactions by date to support real-time queries on recent data.
```

**Code:**

```sql
-- Example SQL statements
CREATE TABLE Transactions (
    transaction_id INT PRIMARY KEY,
    payment_id VARCHAR(50),
    amount DECIMAL(10,2),
    currency_code CHAR(3),
    status VARCHAR(20),
    fraud_score INT,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    CONSTRAINT chk_created_at CHECK (created_at >= DATEADD(day, -30, CURRENT_TIMESTAMP))
)
PARTITION BY RANGE (DATE(created_at));

CREATE TABLE RealTimeTransactions (
    transaction_id INT,
    payment_id VARCHAR(50),
    amount DECIMAL(10,2),
    currency_code CHAR(3),
    status VARCHAR(20),
    created_at TIMESTAMP
);

-- Example of a partitioned table query for real-time fraud scoring
SELECT * FROM Transactions PARTITION (created_at = CURRENT_DATE - INTERVAL '1 DAY');
```

**Example input / output:**

```text
Input:
- Real-time transaction recorded in the `RealTimeTransactions` staging area.

Output:
- Real-time fraud scoring system queries the most recent partition of `Transactions`.
- End-of-day reconciliation processes all transactions from the relevant partitions.
```

**Data:**

```json
{
  "Transaction": {
    "transaction_id": 1,
    "payment_id": "P001",
    "amount": 50.99,
    "currency_code": "USD",
    "status": "completed",
    "fraud_score": 20,
    "created_at": "2023-10-05T14:30:00Z",
    "updated_at": "2023-10-05T14:30:10Z"
  }
}
```

**Flow:**

1. Yes
2. No
3. Real-Time Transaction
4. Staging Area -RealTimeTransactions
5. Is it within the last 30 days?
6. Insert into Transactions
7. Do nothing

