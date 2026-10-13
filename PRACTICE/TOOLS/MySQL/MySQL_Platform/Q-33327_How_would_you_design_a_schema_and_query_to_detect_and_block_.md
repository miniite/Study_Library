---
type: "practice"
difficulty: "Deep"
---

# How would you design a schema and query to detect and block duplicate financial transactions within a short time window (same amount, payer, payee)?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deadlocks in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Deadlocks_in_MySQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of detecting and blocking duplicate financial transactions within a short time window, we need to design an efficient schema and use appropriate queries in MySQL. This involves understanding transaction details like amount, payer, payee, and timestamp, while ensuring that the solution is both performant and scalable.

**Code:**

```sql
-- Example CREATE TABLE statement for transactions
CREATE TABLE Transactions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    amount DECIMAL(10, 2) NOT NULL,
    payer VARCHAR(50) NOT NULL,
    payee VARCHAR(50) NOT NULL,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('PENDING', 'COMPLETED') DEFAULT 'PENDING',
    INDEX idx_amount_payer_payee (amount, payer, payee),
    UNIQUE KEY uk_duplicate_transactions (amount, payer, payee, timestamp)
);
```

**Example input / output:**

```text
-- Example input and output for detecting duplicate transactions within a 5-minute window

-- Inserting sample data
INSERT INTO Transactions (amount, payer, payee) VALUES (100.50, 'Alice', 'Bob');
INSERT INTO Transactions (amount, payer, payee) VALUES (100.50, 'Alice', 'Bob'); -- Duplicate transaction within 5 minutes

-- Query to detect duplicates
SELECT amount, payer, payee, MIN(timestamp), MAX(timestamp)
FROM Transactions
GROUP BY amount, payer, payee
HAVING COUNT(*) > 1;

-- Result: Should return the duplicate transactions and their timestamps
```

**Flow:**

1. Yes
2. No
3. Transaction Insertion
4. Insert into `Transactions` Table
5. Is Amount, Payer, PayeeDuplicate?
6. Mark as Duplicate
7. Persist Transaction as Pending
8. Trigger Block Logic

