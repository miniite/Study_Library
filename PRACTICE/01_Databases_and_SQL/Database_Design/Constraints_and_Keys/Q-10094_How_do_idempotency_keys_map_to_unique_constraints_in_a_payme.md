---
type: "practice"
difficulty: "Deep"
---

# How do idempotency keys map to unique constraints in a payments API, and what happens on a duplicate request that hits the constraint?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Unique and Deferred Constraints](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Unique_and_Deferred_Constraints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a payments API, ensuring the integrity of transactions is critical. One way to achieve this is by leveraging idempotency keys with unique constraints. An idempotency key acts as a unique identifier for each request, ensuring that if the same request is sent multiple times (due to network issues or client retries), it only processes the first instance and ignores subsequent duplicates.

TL;DR: Idempotency keys map to unique constraints by ensuring each payment request is processed exactly once. When a duplicate request arrives with an existing idempotency key, the system should recognize this and reject or ignore the new request to maintain data integrity.

#### Triage

The question asks how idempotency keys relate to unique constraints in a payments API context. The interviewer is probing your understanding of both concepts and their practical application, particularly in handling duplicate requests without violating database constraints. Key assumptions include:

The system uses a relational database.
Idempotency keys are stored as part of the payment request metadata.
Step-by-Step Solution

Understand the Concept:

An idempotency key is essentially a unique identifier for each API request, ensuring that re-sending a request does not result in multiple actions being performed.
A unique constraint ensures no duplicate values are entered into a column.

Mapping Idempotency Keys to Unique Constraints:

In the context of a payments API, the idempotency key can be mapped to a unique constraint on a database field. For instance, you might have a payment_request_id field that is marked as unique in your payments table.
This setup ensures that each payment request can only be processed once.

Handling Duplicate Requests:

When a new payment request arrives with an idempotency key, the system first checks if this key exists in the database.
If the key is found, it means the request has already been processed. The API should either return a success message indicating the operation was already completed or handle the duplicate appropriately (e.g., log the event and ignore the new request).
If the key is not found, the system processes the payment as usual and then stores the idempotency key in the database to prevent future duplicates.

Database Schema Example:

SQL
CREATE TABLE payments (
    id SERIAL PRIMARY KEY,
    amount DECIMAL NOT NULL,
    description TEXT NOT NULL,
    payment_request_id UUID UNIQUE,  -- Unique constraint on this field for idempotency
    status VARCHAR(50) NOT NULL DEFAULT 'PENDING'
);

#### Closing Summary
and Common Mistakes/Follow-Ups
Correctness: Ensuring that duplicate requests are handled correctly is crucial. Incorrect handling can lead to double payments or other financial discrepancies.
Performance: While unique constraints ensure data integrity, they should be optimized for performance, especially in high-load scenarios where many requests may arrive simultaneously.
Failure Modes: Ensure your system has robust retry logic and proper error handling mechanisms in place to manage network failures gracefully.

For follow-ups, the interviewer might ask about implementing distributed systems, handling edge cases like partial request success, or optimizing database indexes for performance.

**Code:**

```sql
CREATE TABLE payments (
    id SERIAL PRIMARY KEY,
    amount DECIMAL NOT NULL,
    description TEXT NOT NULL,
    payment_request_id UUID UNIQUE,  -- Unique constraint on this field for idempotency
    status VARCHAR(50) NOT NULL DEFAULT 'PENDING'
);
```

**Example input / output:**

```

// Example API Request { "amount": 100.00, "description": "Monthly subscription payment", "payment_request_id": "e9d52c68-f3a5-484f-a2cc-ebe487b4dd22" }

CODE
Copy

// Scenario: Duplicate request with the same idempotency key
```json
{
    "amount": 100.00,
    "description": "Monthly subscription payment",
    "payment_request_id": "e9d52c68-f3a5-484f-a2cc-ebe487b4dd22"
}

// Response

JSON
Copy
{
    "status": "PENDING",
    "message": "This request has already been processed."
}
```

**Flow:**

1. Yes
2. No
3. Receive Request
4. Is Idempotency Key Present?
5. Return Success/Log Event
6. Process Payment
7. Store Idempotency Key
8. Return Response: AlreadyProcessed
9. Handle Network Issues/Error

