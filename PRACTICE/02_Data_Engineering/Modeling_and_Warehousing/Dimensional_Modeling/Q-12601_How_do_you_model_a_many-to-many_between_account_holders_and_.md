---
type: "practice"
difficulty: "Deep"
---

# How do you model a many-to-many between account holders and accounts so balances are not double counted when a report groups by holder?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Additive and Allocated Measures](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Additive_and_Allocated_Measures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model a many-to-many relationship between account holders and accounts while ensuring that balances are not double-counted in reports, we need to carefully design our data model using the star or snowflake schema approach. This involves creating an intermediary table (a junction table) to manage the relationships and ensuring consistency in how data is aggregated.

#### Triaging the Question

The core of this question revolves around modeling a many-to-many relationship between account holders and accounts, with the added complexity of preventing double-counting when aggregating balances by holder. The interviewer is probing our understanding of normalization, star/snowflake schemas, and best practices in database design to ensure accurate reporting.

#### First Principles Solution
Identify Entities: We have two primary entities: Account Holders (let's call this table Holders) and Accounts (table Accounts). Each holder can hold multiple accounts, and each account can be held by multiple holders.
Create a Junction Table: To handle the many-to-many relationship, we need an intermediary table that maps between these two entities. This is often called a junction or linking table.
Data Aggregation: When generating reports, we must ensure that balances are aggregated correctly to avoid double-counting.
Step-by-Step Solution

Define Tables:

Holders: Columns include holder_id (primary key), name, etc.
Accounts: Columns include account_id (primary key), balance, currency, etc.
Holder_Accounts: Junction table with columns holder_id and account_id as foreign keys, ensuring a many-to-many relationship.

Insert Data:

SQL
-- Example data for Holders
INSERT INTO Holders (holder_id, name) VALUES (1, 'Alice');
INSERT INTO Holders (holder_id, name) VALUES (2, 'Bob');

-- Example data for Accounts
INSERT INTO Accounts (account_id, balance, currency) VALUES (101, 500.00, 'USD');
INSERT INTO Accounts (account_id, balance, currency) VALUES (102, 750.00, 'USD');

-- Example data for Holder_Accounts
INSERT INTO Holder_Accounts (holder_id, account_id) VALUES (1, 101);
INSERT INTO Holder_Accounts (holder_id, account_id) VALUES (1, 102);

Aggregating Data: To ensure that balances are not double-counted when generating reports by holder, we can use SQL joins and aggregate functions.

SQL Query for Aggregation:

SQL
SELECT H.name,
       SUM(A.balance) AS total_balance
  FROM Holders H
  JOIN Holder_Accounts HA ON H.holder_id = HA.holder_id
  JOIN Accounts A ON HA.account_id = A.account_id
 GROUP BY H.name;

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution ensures that the many-to-many relationship between account holders and accounts is properly modeled, preventing double-counting of balances in reports. Key points to remember are:

Using a junction table effectively.
Ensuring proper normalization.
Aggregating data correctly using SQL joins.

#### Common mistakes
include failing to use a junction table or not properly aggregating the data, leading to incorrect counts and potential bugs in the reporting system. Follow-up questions might explore the impact on performance, the need for additional indexes, and how this model scales with more complex relationships and larger datasets.

**Comparison:**

| Column Name | Data Type    |
|-------------|-------------|
| holder_id   | INT         |
| name        | VARCHAR(255)|
| account_id  | INT         |
| balance     | DECIMAL(10,2)|
| currency    | VARCHAR(3)  |

**Code:**

```sql
-- Example SQL code for creating tables
CREATE TABLE Holders (
    holder_id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE Accounts (
    account_id INT PRIMARY KEY,
    balance DECIMAL(10, 2),
    currency VARCHAR(3)
);

CREATE TABLE Holder_Accounts (
    holder_id INT,
    account_id INT,
    FOREIGN KEY (holder_id) REFERENCES Holders(holder_id),
    FOREIGN KEY (account_id) REFERENCES Accounts(account_id)
);
```

**Example input / output:**

```text
-- Sample SQL query for aggregation
SELECT H.name,
       SUM(A.balance) AS total_balance
FROM Holders H
JOIN Holder_Accounts HA ON H.holder_id = HA.holder_id
JOIN Accounts A ON HA.account_id = A.account_id
GROUP BY H.name;
```

**Data:**

```json
{
  "holders": [
    { "holder_id": 1, "name": "Alice" },
    { "holder_id": 2, "name": "Bob" }
  ],
  "accounts": [
    { "account_id": 101, "balance": 500.00, "currency": "USD" },
    { "account_id": 102, "balance": 750.00, "currency": "USD" }
  ],
  "holder_accounts": [
    { "holder_id": 1, "account_id": 101 },
    { "holder_id": 1, "account_id": 102 },
    { "holder_id": 2, "account_id": 101 }
  ]
}
```

**Flow:**

1. Account Holders
2. Holders
3. Accounts
4. Holder_Accounts
5. Accounts
6. Holders
7. Holder_Accounts
8. Accounts
9. Aggregation Query
10. Total Balance by Holder

