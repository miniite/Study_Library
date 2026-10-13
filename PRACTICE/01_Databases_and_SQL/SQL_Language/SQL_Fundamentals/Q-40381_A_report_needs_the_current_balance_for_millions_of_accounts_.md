---
type: "practice"
difficulty: "Deep"
---

# A report needs the current balance for millions of accounts derived from a transactions table. Compare recomputing from events, a maintained balance column with triggers, and periodic snapshots. Which and why?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Replica Staleness](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Replica_Staleness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compare the approaches for computing the current balance of millions of accounts—recomputing from events, maintaining a balance column with triggers, and using periodic snapshots—we need to consider several factors such as performance, consistency, accuracy, and operational complexity. Let's break down each approach step by step.

#### Triage
Understanding the Problem

The question asks us to compare three methods for calculating the current account balances:

Recomputing from events: Recalculating balance based on transaction history.
Maintained balance column with triggers: Keeping a running total in a separate column updated via database triggers.
Periodic snapshots: Taking periodic backups or snapshots of the account balances.

#### Approach
Recomputing from Events

#### Triage
This approach involves querying the transactions table to compute the balance by summing up all relevant transactions for each account at any given time.

#### Assumptions
The system handles a high volume of transactions and requires real-time or near-real-time balance calculations.

#### Trade-offs

#### Pros
Accurate if the transaction history is fully captured and processed.

#### Cons
Performance overhead, especially for large datasets; each request to compute balance involves querying the transactions table.

#### Approach
Maintained Balance Column with Triggers

#### Triage
This method relies on maintaining a running total of account balances in a separate column that gets updated automatically via database triggers whenever a transaction occurs.

#### Assumptions
The system supports database features like triggers and has sufficient performance to handle updates without significant degradation.

#### Trade-offs

#### Pros
Near real-time balance calculations; no need to recompute from events every time.
Reduced querying overhead as balances are stored directly in the accounts table.

#### Cons
Increased complexity of maintaining triggers and ensuring their correctness.
Potential for latency if transactions are not immediately reflected in the balance column.

#### Approach
Periodic Snapshots

#### Triage
This approach involves periodically taking snapshots of account balances, which can be queried to get current balances.

#### Assumptions
The system supports snapshotting functionality and has a relatively stable transaction volume.

#### Trade-offs

#### Pros
Balance calculations are fast as they involve querying precomputed values stored in the snapshots.
Simplifies balance calculation logic.

#### Cons
Inaccurate balances if there have been transactions since the last snapshot was taken.
Increased storage and operational overhead for maintaining multiple snapshot versions.

#### First-Principles Solution

Recomputing from Events:

SQL
SELECT account_id, SUM(amount) AS current_balance
FROM transactions
WHERE transaction_date >= '2023-10-01'
GROUP BY account_id;
This method is accurate but performance-intensive for large datasets.

Maintained Balance Column with Triggers:

SQL
CREATE TABLE accounts (
  account_id INT PRIMARY KEY,
  current_balance DECIMAL(10, 2),
  -- Other fields...
);

CREATE TRIGGER balance_update AFTER INSERT ON transactions FOR EACH ROW BEGIN
  UPDATE accounts SET current_balance = current_balance + NEW.amount WHERE account_id = NEW.account_id;
END;
This approach ensures real-time accuracy and reduces query overhead but introduces complexity in maintaining triggers.

Periodic Snapshots:

SQL
-- Create a snapshot table for historical balance data.
CREATE TABLE balance_snapshots (
  id INT AUTO_INCREMENT PRIMARY KEY,
  account_id INT,
  snapshot_date DATE,
  current_balance DECIMAL(10, 2),
  FOREIGN KEY (account_id) REFERENCES accounts(account_id)
);

-- Periodic job to create snapshots:
INSERT INTO balance_snapshots (account_id, snapshot_date, current_balance)
SELECT account_id, CURDATE(), SUM(amount)
FROM transactions
WHERE transaction_date >= 'start_of_snapshot_interval'
GROUP BY account_id;
This method simplifies query performance but sacrifices accuracy for timeliness.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the choice of approach depends on the specific requirements:

Recomputing from Events is best suited for systems that can handle high transaction volumes without significant latency concerns.
Maintained Balance Column with Triggers provides a good balance between accuracy and performance but requires careful maintenance to avoid issues like data staleness.
Periodic Snapshots offers the fastest queries but at the cost of potential inaccuracies unless snapshots are taken frequently.

#### Common mistakes
include:

Not considering the trade-offs in terms of performance, complexity, and accuracy.
Overlooking the need for robust error handling and consistency checks, especially with triggers.
Failing to manage storage and snapshot overhead effectively.

#### Follow-ups
might explore how these methods handle edge cases such as concurrent transactions, transaction rollbacks, and data integrity constraints.

