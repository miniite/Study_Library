---
type: "practice"
difficulty: "Expert"
---

# How do you audit a large existing codebase to find every transaction whose correctness silently depends on the current default isolation level? What tooling would you build?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Setting Isolation Levels](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

Auditing a large codebase for transactions that depend on specific isolation levels requires understanding how different isolation levels (RU, RC, RR, Serializable) affect transaction behavior and identifying cases where silent correctness issues might arise. The key is to systematically traverse the codebase, analyze transactions, and instrument or build custom tools to detect these dependencies.

#### Triaging the Question

The question is asking about a systematic approach to identify all transactions in an existing database application that rely on the current default isolation level for their correct behavior. This involves understanding how different isolation levels (Read Uncommitted, Read Committed, Repeatable Read, and Serializable) affect transaction outcomes. The goal is to find instances where silent correctness issues might occur if the default isolation level changes.

#### Assumptions
The codebase is written in a language that supports explicit transactions.
The current default isolation level can be changed without affecting application behavior (e.g., it's not embedded in SQL queries).
Transactions are identified by specific patterns or boundaries within the code.
Step-by-Step Solution:
1. Understand Isolation Levels:
Read Uncommitted: Allows dirty reads, where uncommitted transactions can be read.
Read Committed: Prevents dirty reads but allows non-repeatable reads and phantom reads.
Repeatable Read: Prevents non-repeatable reads and phantom reads but may still allow lost updates.
Serializable: Ensures all transactions are serialized, preventing any concurrency issues.
2. Identify Transaction Boundaries:
Traverse the codebase to identify transaction boundaries using context managers or explicit BEGIN, COMMIT, ROLLBACK statements in SQL queries.
Use static analysis tools like SonarQube with plugins for database transactions or custom scripts to scan source code.
3. Analyze Transaction Logic:
For each transaction boundary, analyze the logic inside:
Check if it relies on a specific order of operations that might be disrupted by different isolation levels.
Identify reads and writes within transactions to understand their dependencies.
Look for shared resources (tables, records) accessed within multiple transactions.
4. Instrumentation or Custom Tooling:
Logging: Introduce logging at key points in the application to capture transaction behavior before and after changes in isolation levels.
PYTHON
from db_utils import execute_query

def log_transaction(transaction_id):
    start_time = time.time()
    result = execute_query(f"SELECT * FROM some_table WHERE id = {transaction_id}")
    end_time = time.time()
    logging.info(f"Transaction {transaction_id} took {end_time - start_time:.2f} seconds")
Custom Analyzers: Build a custom tool using database-specific APIs or ORM layer to analyze transaction behavior.
PYTHON
from sqlalchemy.orm import Session

class TransactionAnalyzer:
    def __init__(self, session: Session):
        self.session = session

    def audit_transactions(self, transactions):
        for tx in transactions:
            # Perform analysis on each transaction
            pass
5. Automate Testing with Different Isolation Levels:
Run automated tests with different isolation levels to simulate the impact.
PYTHON
from sqlalchemy import create_engine

engine = create_engine('sqlite:///test.db', isolation_level='READ_UNCOMMITTED')
with Session(engine) as session:
    # Execute test cases here
6. Review and Refactor:
After auditing, review findings and refactor code where necessary to ensure correct behavior across different isolation levels.
Document changes and update the application's deployment strategy if needed.

#### Closing Summary

By understanding the impact of different isolation levels on transactions, systematically identifying transaction boundaries, analyzing logic, instrumenting the codebase, and running automated tests, you can effectively audit a large existing codebase to find every transaction whose correctness depends on the current default isolation level. This process ensures that changes in isolation settings do not silently break application behavior.

#### Common Mistakes/Follow-Ups
Over-reliance on Default Levels: Not all transactions need high isolation levels, leading to unnecessary overhead.
Inadequate Testing: Failing to run tests with different isolation levels can miss critical dependencies.
Manual Audits: Assuming that manual audits alone are sufficient without using tools for automation.

**Comparison:**

```
Transaction ID	Operations	Dependencies
1	SELECT, UPDATE, SELECT	tableA
2	INSERT, SELECT, DELETE	tableB
```

**Code:**

```python
from db_utils import execute_query

def log_transaction(transaction_id):
    start_time = time.time()
    result = execute_query(f"SELECT * FROM some_table WHERE id = {transaction_id}")
    end_time = time.time()
    logging.info(f"Transaction {transaction_id} took {end_time - start_time:.2f} seconds")
```

**Example input / output:**

```text
Input:
- Codebase with explicit transaction boundaries

Output:
- List of transactions and their dependencies for different isolation levels
- Log files containing transaction performance metrics
```

**Data:**

```json
{
  "transactions": [
    {
      "id": 1,
      "operations": ["SELECT", "UPDATE", "SELECT"],
      "dependencies": ["tableA"]
    },
    {
      "id": 2,
      "operations": ["INSERT", "SELECT", "DELETE"],
      "dependencies": ["tableB"]
    }
  ]
}
```

**Flow:**

1. Identify TransactionBoundaries
2. Analyze Logic
3. Audit Transactions withCustom Tools
4. Automate Testing withDifferent Isolation Levels
5. Refactor and DocumentChanges

