---
type: "practice"
difficulty: "Expert"
---

# How do you propagate a deadline (not just a timeout) through the gateway so that if a client has 200ms left, each downstream hop shrinks its own budget accordingly instead of every hop starting a fresh timeout?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Timeouts Retries and Backoff](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Timeouts_Retries_and_Backoff.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In this scenario, we need to propagate a deadline (not just a timeout) through an API Gateway so that downstream services respect the client's remaining time budget. This involves understanding how timeouts and deadlines are handled in requests, leveraging asynchronous programming, and implementing custom middleware to ensure accurate propagation.

#### Triage
The question revolves around managing request lifecycles and ensuring that each service in the chain respects a given deadline instead of starting fresh timers. The interviewer is probing your knowledge of request handling, asynchronous operations, and how to maintain state across multiple hops. Key assumptions include the use of HTTP requests, middleware for handling deadlines, and potentially async/await or promises for managing timeouts.

#### First-Principles Solution

Understand Deadlines vs. Timeouts: A deadline is a point in time by which a task must be completed, whereas a timeout is an action that occurs if the task hasn't been completed within a certain period.
Maintain State Across Hops: To propagate the deadline correctly, we need to ensure each service knows about the remaining time budget and acts accordingly.
Use Middleware for Deadline Propagation: Implement middleware that captures the deadline from the request and passes it down the stack.

Here’s how you can implement this:

1. Capture the Deadline

When a request enters the gateway, capture the deadline provided by the client.

JAVASCRIPT
function captureDeadline(req) {

#### cons
t deadline = req.header("X-Client-DL"); // Example header for the deadline
    return deadline ? Date.parse(deadline) : null;
}
2. Middleware for Propagation

Create middleware to propagate the deadline through each hop.

JAVASCRIPT
async function handleDeadline(req, res, next) {

#### cons
t originalDeadline = captureDeadline(req);

    // If there's no deadline, pass through without modification.
    if (!originalDeadline) return next();

    req.deadline = originalDeadline;
    next();
}

function propagateDeadline(next) {
    return async (req, res, next) => {

#### cons
t remainingTime = Date.now() - req.startTime; // Measure elapsed time

#### cons
t deadlineBudget = req.deadline - remainingTime;

        if (deadlineBudget <= 0) { // Time's up!
            return res.status(503).send("Deadline exceeded");
        }

        req.deadline = deadlineBudget;
        next();
    };
}
3. Apply Middleware to Each Service

Ensure that each downstream service respects the remaining time budget.

JAVASCRIPT

#### cons
t services = [serviceA, serviceB, serviceC];
services.forEach(service => {

#### cons
t serviceWrapper = propagateDeadline(handleDeadline)(service);
});

app.use("/api", services);
4. Downstream Services Respect Deadline

Each downstream service should respect the deadline and measure its own time budget accurately.

JAVASCRIPT
async function serviceA(req, res) {
    // Simulate some processing
    await delay(50); // Simulated operation that takes 50ms

#### cons
t remainingTime = req.deadline - Date.now();

    if (remainingTime <= 0) { // Time's up!
        return res.status(503).send("Deadline exceeded");
    }

    // Continue processing...
}

async function delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

#### Closing Summary

By using middleware to capture and propagate the deadline, each service can respect the client's remaining time budget. This ensures that the entire request lifecycle respects deadlines rather than starting fresh timers at each hop.

#### Common Mistakes/Follow-Ups

Incorrect Deadline Calculation: Ensure you correctly calculate the remaining time budget.
Ignoring Client-Side Errors: Make sure to handle cases where the client fails to set a deadline properly.
Error Handling: Implement robust error handling for timeouts and deadlines exceeding.

#### Performance Considerations
Evaluate if custom middleware adds unnecessary overhead, especially in high-throughput environments.

**Comparison:**

```
Service	Processing Time (ms)
A	50
B	100
C	75
```

**Code:**

```javascript
function captureDeadline(req) {
    const deadline = req.header("X-Client-DL"); // Example header for the deadline
    return deadline ? Date.parse(deadline) : null;
}

async function handleDeadline(req, res, next) {
    const originalDeadline = captureDeadline(req);

    if (!originalDeadline) return next();

    req.deadline = originalDeadline;
    next();
}

function propagateDeadline(next) {
    return async (req, res, next) => {
        const remainingTime = Date.now() - req.startTime; // Measure elapsed time
        const deadlineBudget = req.deadline - remainingTime;

        if (deadlineBudget <= 0) { // Time's up!
            return res.status(503).send("Deadline exceeded");
        }

        req.deadline = deadlineBudget;
        next();
    };
}

const services = [serviceA, serviceB, serviceC];
services.forEach(service => {
    const serviceWrapper = propagateDeadline(handleDeadline)(service);
});

app.use("/api", services);

async function serviceA(req, res) {
    await delay(50); // Simulated operation that takes 50ms

    const remainingTime = req.deadline - Date.now();

    if (remainingTime <= 0) { // Time's up!
        return res.status(503).send("Deadline exceeded");
    }

    // Continue processing...
}

async function delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}
```

**Example input / output:**

```text
Input:
- Client request with header "X-Client-DL" set to a specific deadline.
- Each service has predefined processing times.

Output:
- If the total processing time across services exceeds the client's deadline, return a 503 error indicating "Deadline exceeded".
- Otherwise, continue processing and complete the request within the remaining budget of the deadline.
```

**Data:**

```json
{
    "req": {
        "header": {"X-Client-DL": "2023-10-05T14:30:00Z"}
    },
    "services": [
        { "name": "serviceA", "processingTime": 50 },
        { "name": "serviceB", "processingTime": 100 },
        { "name": "serviceC", "processingTime": 75 }
    ]
}
```

**Flow:**

1. Capture Deadline
2. Handle Deadline
3. Propagate Deadline
4. Service A (SimulatedOperation)
5. Service B (SimulatedOperation)
6. Service C (SimulatedOperation)

