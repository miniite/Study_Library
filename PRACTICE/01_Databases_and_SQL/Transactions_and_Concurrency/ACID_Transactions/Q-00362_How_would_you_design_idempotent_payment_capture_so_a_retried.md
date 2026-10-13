---
type: "practice"
difficulty: "Deep"
---

# How would you design idempotent payment capture so a retried webhook never double-charges a customer?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Atomicity and Money Transfers](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Atomicity_and_Money_Transfers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an idempotent payment capture system, ensuring that a retried webhook does not result in double-charging a customer, we need to ensure atomicity, consistency, isolation, and durability (ACID properties) within our transactional processes. This involves handling retries in a way that respects the database's constraints on transactions.

In essence, we are looking for a mechanism that ensures idempotency at both the application level and the database level. Let’s break down the solution from first principles:

Understand What We Are Probing: The interviewer is likely probing your understanding of transactional properties, distributed systems concepts (like retries), and how to ensure data integrity in a highly available system.

#### Assumptions

We are working with a relational database.
Webhooks are used for capturing payments asynchronously.
Retries might be due to network issues or server failures.
The goal is to prevent double-charging, even if the webhook is retried multiple times.

Step-by-Step Solution:

Identify Key Entities: We need a Payment entity with fields like payment_id, customer_id, and status.
Database Constraints: Use database constraints to ensure data integrity.
Application-Level Logic: Implement idempotent logic in the application layer.
Step-by-Step Solution

Create Payment Entity:

SQL
CREATE TABLE Payments (
    payment_id BIGINT PRIMARY KEY,
    customer_id VARCHAR(255),
    status ENUM('CREATED', 'PAID', 'FAILED') DEFAULT 'CREATED',
    amount DECIMAL(10, 2)
);

Ensure Atomicity and Durability:

Use transactions to ensure atomicity.
Ensure the database is durable by committing only after a successful payment.

Implement Idempotent Logic in Application:

When a webhook is received, check if the Payment exists with status = 'CREATED'.
If it does, update its status to PAID and commit the transaction.
If it doesn't exist or has already been paid, return an appropriate response.

Use UUIDs for Payment IDs:

Ensure each payment is uniquely identifiable using a UUID.
This helps in tracking retries accurately.

Database-Level Constraints:

Add unique constraints on customer_id and amount to prevent duplicate payments accidentally.

Retry Logic with Exponential Backoff:

Implement retry logic at the application level, possibly using an exponential backoff strategy.
Use a queue (e.g., Kafka) for handling retries to ensure messages are not lost on failure.

#### Example
Workflow
PYTHON
def handle_webhook(webhook_data):
    payment_id = webhook_data['payment_id']
    customer_id = webhook_data['customer_id']

    with session.begin():
        # Check if the payment is in a CREATED state
        existing_payment = session.query(Payment).filter(
            Payment.payment_id == payment_id,
            Payment.status == 'CREATED'
        ).first()

        if not existing_payment:
            return "Payment already processed"

        # Mark as paid and commit transaction
        existing_payment.status = 'PAID'
        session.commit()

    return "Payment successfully captured"

#### Closing Summary

By ensuring that the payment process respects ACID properties, we can design a robust system that handles retries without double-charging customers. This involves a combination of database constraints, application-level logic, and retry strategies.

#### Common Mistakes/Follow-Ups

Forgetting to check the status before processing.
Not using transactions or ensuring proper commit semantics.
Ignoring database uniqueness constraints.
Failing to implement proper exponential backoff for retries.

**Comparison:**

```
payment_id	customer_id	status	amount
123456	customer-1	PAID	10.00
```

**Code:**

```python
def handle_webhook(webhook_data):
    payment_id = webhook_data['payment_id']
    customer_id = webhook_data['customer_id']

    with session.begin():
        existing_payment = session.query(Payment).filter(
            Payment.payment_id == payment_id,
            Payment.status == 'CREATED'
        ).first()

        if not existing_payment:
            return "Payment already processed"

        existing_payment.status = 'PAID'
        session.commit()

    return "Payment successfully captured"
```

**Example input / output:**

```

Input:

JSON
Copy
{
    "payment_id": 123456,
    "customer_id": "customer-1"
}

Output:

CODE
Copy
Payment successfully captured
```

**Data:**

```sql
CREATE TABLE Payments (
    payment_id BIGINT PRIMARY KEY,
    customer_id VARCHAR(255),
    status ENUM('CREATED', 'PAID', 'FAILED') DEFAULT 'CREATED',
    amount DECIMAL(10, 2)
);
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Webhook Received] --> B{Payment Exists?};
4. B -->|No| C[Paid];
5. B -->|Yes| D[Mark as Paid & Commit];
6. C --> E[Return "Payment Already Processed"]
7. D --> F[Return "Payment Successfully Captured"];

