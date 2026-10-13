---
type: "practice"
difficulty: "Deep"
---

# How would you unit-test an LLD that depends on the current time, random shuffling, and a payment gateway; walk through the seams (clock, RNG, gateway) you'd design in so tests are deterministic.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Open-closed Extensibility](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Open-closed_Extensibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To unit-test an LLD that relies heavily on the current time, randomness, and external services like a payment gateway, we need to isolate these dependencies and make our tests deterministic. The key idea is to replace real-time calls with mocks or stubs so we can control the state during testing. This involves designing seams in the code for each dependency—like using an in-memory clock, a fixed random number generator (RNG), and a mock payment gateway.

Let's walk through how we would set up these seams step by step:

Current Time Dependency: The first dependency is the current time. We need to mock this out because it changes continuously and unpredictably. To make our tests deterministic, we'll use an in-memory clock that can be controlled during testing.

Random Number Generation: Randomness introduces another layer of unpredictability into the system. We should replace any calls to a random number generator (RNG) with a fixed sequence or controlled randomness within tests.

Payment Gateway Dependency: Payment gateways are often external services, and testing them directly can be problematic due to latency and failure modes. Instead, we create mock versions of the payment gateway that mimic its behavior in predictable ways.

Now, let's dive into how these seams would look in practice.

#### Triage
What We're Probing

The interviewer is probing your ability to:

Identify and isolate dependencies.
Use mocks/stubs effectively to control external states.
Design for testability by creating clear seams between the core logic and its environment.
Understand the trade-offs of deterministic testing versus real-world unpredictability.

#### First-Principles Solution
1. Clock Seam

We use an in-memory clock that can be controlled during tests. For example, in Node.js or Java, we might create a mock time service like this:

JAVASCRIPT
// Example in JavaScript
class MockTimeService {
  currentTime = Date.now();

  advanceTo(timestamp) {
    this.currentTime = timestamp;
  }

  getCurrentTime() {
    return new Date(this.currentTime);
  }
}

In tests, we can set the advanceTo method to a specific point in time and verify that our system behaves as expected.

2. Random Number Generation Seam

For randomness, we replace calls to the real RNG with a controlled sequence or fixed values. This could be done by wrapping the random number generator with a mock object:

JAVASCRIPT
// Example in JavaScript
class MockRandomNumberGenerator {

#### cons
tructor(seed) {
    this.value = seed;
  }

  next() {
    return this.value++;
  }
}

We can then instantiate and use this mock RNG to produce consistent sequences during tests.

3. Payment Gateway Seam

For the payment gateway, we create a mock service that simulates its behavior. This might include handling credit card validation, fraud checks, etc., with predictable outcomes:

JAVASCRIPT
// Example in JavaScript
class MockPaymentGateway {
  chargeCard(cardDetails) {
    // Simulate successful or failed transaction based on predefined rules
    if (cardDetails.cardNumber === "4111111111111111") return true;
    return false;
  }
}

In the actual system, we would replace this mock with a real payment gateway service, but during tests, it allows us to control responses and verify that our logic handles transactions correctly.

#### Closing Summary

By isolating dependencies through these seams, we can create deterministic tests that are repeatable and consistent. This approach ensures that our tests do not rely on external factors like the current time or real-world behavior of third-party services, making them more reliable and easier to maintain.

#### Common Mistakes/Follow-Ups

Forgetting to mock all dependencies fully.
Not considering edge cases in random number generation (e.g., sequences and boundary conditions).
Overlooking the importance of test isolation vs. integration testing balance.
Depth + Language Matching

Given that this is a deep understanding question, we need to demonstrate not just how to implement these seams but also why they are important and discuss trade-offs. For instance, while deterministic tests are beneficial for stability, there might be cases where real-world unpredictability is crucial (e.g., performance under load).

**Comparison:**

```
Time	Random Number	Payment Result
1628504000	1	True
1628504001	2	False
```

**Code:**

```javascript
// MockTimeService example in JavaScript
class MockTimeService {
  currentTime = Date.now();

  advanceTo(timestamp) {
    this.currentTime = timestamp;
  }

  getCurrentTime() {
    return new Date(this.currentTime);
  }
}

// MockRandomNumberGenerator example in JavaScript
class MockRandomNumberGenerator {
  constructor(seed) {
    this.value = seed;
  }

  next() {
    return this.value++;
  }
}

// MockPaymentGateway example in JavaScript
class MockPaymentGateway {
  chargeCard(cardDetails) {
    // Simulate successful or failed transaction based on predefined rules
    if (cardDetails.cardNumber === "4111111111111111") return true;
    return false;
  }
}
```

**Example input / output:**

```text
Test Input:
- MockTimeService is set to a specific timestamp.
- MockRandomNumberGenerator sequence is predefined.
- MockPaymentGateway returns a known response.

Expected Output:
- System logic processes the request correctly based on the controlled time, random numbers, and payment status.
```

**Data:**

```json
{
  "cardDetails": {
    "cardNumber": "4111111111111111",
    "expiryDate": "06/23"
  },
  "paymentResult": true
}
```

**Flow:**

1. MockTimeService
2. MockRandomNumberGenerator
3. MockPaymentGateway
4. User
5. Request Processing Logic
6. Dependency on Clock
7. (Clock Controlled)
8. Dependency on RNG
9. (Random Numbers Controlled)
10. Dependency on PaymentGateway
11. (Payment ResponsesControlled)
12. System Logic BehaviorVerification
13. Expected OutcomeDetermined

