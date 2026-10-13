---
type: "practice"
difficulty: "Deep"
---

# You must set an isolation policy for a system where some transactions are safe to retry and some are not (external side effects). How do you architect the boundary and enforce it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Setting Isolation Levels](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting an isolation policy in a database, we need to balance transactional consistency with system resilience. In this scenario, we have transactions that are safe to retry and others that should not be retried due to external side effects. The key is to architect the boundaries carefully to enforce these policies while ensuring data integrity.

First, let's clarify what isolation levels (RU/RC/RR/Serializable) mean:

RU (Read Uncommitted): No locks are placed, but dirty reads can occur.
RC (Read Committed): Ensures that transactions cannot read uncommitted changes from other transactions.
RR (Repeatable Read): Ensures the same data is always returned by a query in the same transactional context.
Serializable: Ensures that transactions are isolated, like serial execution.

For our system, we need to use a combination of isolation levels and boundary enforcement. Here’s how we can approach it:

Define Transaction Categories:

Safe to Retry (S2R): These transactions can be retried without causing external side effects.
Not Safe to Retry (NS2R): These transactions must not be retried due to potential external side effects.

Architectural Boundaries:

Implement a middleware layer that acts as an entry point for all database access.
This layer will enforce the isolation policy based on transaction type.

Policy Enforcement:

For S2R transactions, use higher isolation levels like RR or Serializable to ensure data consistency.
For NS2R transactions, use RC to avoid issues with uncommitted changes but ensure they are safe from external side effects.

Boundary Conditions:

Ensure that all S2R transactions pass through the middleware layer.
Implement a retry mechanism for failed S2R transactions without causing conflicts.

Now let’s delve into the implementation details:

#### First-Principles Solution

Middleware Layer:

Create a service layer that abstracts database interactions.
This layer will categorize each transaction based on whether it is safe to retry or not.

Transaction Categorization:

Use metadata or tags in the request headers to identify if a transaction is S2R or NS2R.
For example, use HTTP headers like X-Transaction-Type with values such as s2r or ns2r.

Isolation Level Selection:

Based on the transaction type, select the appropriate isolation level.
If X-Transaction-Type: s2r, set isolation to RR or Serializable.
If X-Transaction-Type: ns2r, set isolation to RC.

Retry Mechanism:

Implement a retry mechanism for S2R transactions that retries only after ensuring no external side effects have occurred.
This can be done by checking external systems (e.g., APIs) for any changes or using a timestamp-based approach to detect conflicts.

#### Example
Code
PYTHON
def handle_transaction(request):
    transaction_type = request.headers.get('X-Transaction-Type')

    if transaction_type == 's2r':
        isolation_level = 'RR'
        retry_policy = RetryPolicy(max_retries=3, delay_seconds=5)
    elif transaction_type == 'ns2r':
        isolation_level = 'RC'
        retry_policy = None

    # Execute the transaction with the selected isolation level
    execute_transaction(isolation_level)

    return response

def execute_transaction(isolation_level):
    with DatabaseConnection.isolation_level(isolation_level):
        try:
            # Perform database operations
            pass
        except DatabaseError as e:
            if retry_policy and not is_external_side_effect_occurred():
                retry_policy.retry()
            else:
                raise e

#### Closing Summary

In summary, we need to architect a middleware layer that categorizes transactions based on their safety for retries. This layer will enforce the appropriate isolation levels (RU/RC/RR/Serializable) and implement a retry mechanism for safe-to-retry transactions while ensuring no external side effects occur.

#### Common Mistakes

Failing to distinguish between S2R and NS2R transactions.
Not implementing proper error handling and retry mechanisms.
Incorrectly setting isolation levels, leading to either too much or too little transactional consistency.

