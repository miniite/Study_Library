---
type: "practice"
difficulty: "Intermediate"
---

# Describe how you'd instrument code to attach a correlation id so every deadlock log ties back to a specific request and user action.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlock Causes and Diagnosis](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Deadlock_Causes_and_Diagnosis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that every deadlock log can be traced back to a specific request and user action, we need to implement an approach that attaches a unique correlation ID to each database transaction or request. This will help in correlating logs and diagnosing issues more effectively.

#### Triage

The interviewer is probing your understanding of logging practices, especially in the context of complex operations like deadlock detection. They are likely interested in how you would ensure traceability, manage state consistency across multiple transactions, and handle potential performance impacts.

Key Assumptions:
We have a multi-threaded or distributed application that interacts with a database.
Deadlocks can occur due to resource contention between concurrent operations.
Each transaction or request has an associated user ID for tracing purposes.

#### First-Principles Solution
Step 1: Define the Correlation ID

A correlation ID is typically a unique identifier assigned to each request or operation, which is passed through all layers of the application stack. This can be achieved using middleware like Express.js in Node.js applications, interceptors in Spring Boot Java apps, or custom middleware functions.

JAVASCRIPT
// Example with Express.js Middleware
app.use((req, res, next) => {

#### cons
t correlationId = uuidv4(); // Generate a unique ID
  req.correlationId = correlationId;
  res.setHeader('X-Correlation-ID', correlationId); // Send the ID in headers for logging
  next();
});
Step 2: Attaching Correlation ID to Database Transactions

Each database transaction should carry this correlation ID. This can be done using connection pooling libraries that support custom metadata or by manually setting up a connection with the session.

JAVASCRIPT

#### cons
t { Pool } = require('pg');

#### cons
t pool = new Pool({ ... });

// Using pg-promise library for Postgres

#### cons
t pgp = require('pg-promise')();

#### cons
t db = pgp({
  connectionString: process.env.DATABASE_URL,
  statement: (sql, values) => {
    return { sql, values, correlationId: req.correlationId }; // Attach the ID to each SQL execution
  }
});

async function someFunction(req) {

#### cons
t result = await db.one(/* SQL query */, [/* params */]);

#### cons
ole.log(result);
}
Step 3: Logging Deadlocks with Correlation ID

When a deadlock is detected, it should log this information along with the correlation ID. This can be done by intercepting deadlocks using database-specific mechanisms or custom error handling.

JAVASCRIPT
try {

#### cons
t result = await db.one(/* SQL query */, [/* params */]);
} catch (err) {
  if (err.code === '40P01' || err.code === '53310') { // Deadlock lock violation codes in PostgreSQL

#### cons
ole.error(`Deadlock detected with correlation ID: ${req.correlationId}`);
    logger.error({ correlationId: req.correlationId }, 'Deadlock occurred');
  }
}
Step 4: Ensuring Consistency and Performance

Ensure that the overhead of attaching a correlation ID is minimal. Use efficient UUID generators like uuidv1 or uuidv4. Also, consider batching log entries to reduce the number of writes.

#### Closing Summary

By implementing this approach, we can ensure that every deadlock event logs are tied back to specific user actions and requests, facilitating easier debugging and performance analysis.

#### Common Mistakes/Follow-Ups
Not handling all edge cases: Ensure you cover scenarios where transactions might fail before logging.
Overhead concerns: Be mindful of the performance impact of adding correlation IDs. Use efficient UUID generators and batch log entries when possible.
Missing error handling: Make sure to handle deadlocks gracefully without crashing the application.
Inconsistent log format: Ensure logs are formatted consistently for easier analysis.

**Comparison:**

```
Transaction	User Action	Deadlock Log
T001	Insert	CorrelationID: abc123def456, Deadlock Occurred
```

**Code:**

```javascript
// Example with Express.js Middleware
app.use((req, res, next) => {
  const correlationId = uuidv4(); // Generate a unique ID
  req.correlationId = correlationId;
  res.setHeader('X-Correlation-ID', correlationId); // Send the ID in headers for logging
  next();
});

// Using pg-promise library for Postgres
const pgp = require('pg-promise')();
const db = pgp({
  connectionString: process.env.DATABASE_URL,
  statement: (sql, values) => {
    return { sql, values, correlationId: req.correlationId }; // Attach the ID to each SQL execution
  }
});

async function someFunction(req) {
  const result = await db.one(/* SQL query */, [/* params */]);
  console.log(result);
}

try {
  const result = await db.one(/* SQL query */, [/* params */]);
} catch (err) {
  if (err.code === '40P01' || err.code === '53310') { // Deadlock lock violation codes in PostgreSQL
    console.error(`Deadlock detected with correlation ID: ${req.correlationId}`);
    logger.error({ correlationId: req.correlationId }, 'Deadlock occurred');
  }
}
```

**Example input / output:**

```text
Input:
Request to execute SQL query with correlation ID: abc123def456

Output:
SQL query executed successfully.

Error (Deadlock):
Deadlock detected with correlation ID: abc123def456
```

**Data:**

```json
{
  "correlation_id": "abc123def456"
}
```

**Flow:**

1. Yes
2. Success
3. Deadlock
4. Start
5. Is Correlation ID Required?
6. Generate UUID for CorrelationID
7. Attach Correlation ID toRequest/Transaction
8. Database Transaction
9. Log Success with CorrelationID
10. Log Deadlock with CorrelationID and Error Details
11. End

