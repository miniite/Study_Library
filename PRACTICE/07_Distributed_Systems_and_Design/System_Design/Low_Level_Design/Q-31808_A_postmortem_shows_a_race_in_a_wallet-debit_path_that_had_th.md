---
type: "practice"
difficulty: "Expert"
---

# A postmortem shows a race in a wallet-debit path that had "thread-safe" comments but a broken check-then-act; as principal how do you change the org's approach so correctness of concurrency is enforced by design (types/APIs) rather than by reviewer vigilance?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Lock Granularity Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To enforce correctness in concurrency, we need to shift our focus from relying on thread-safety comments and manual checks to a more robust design approach using strong typing and well-defined APIs. By leveraging Rust's ownership model or similar static type systems, we can ensure that the program’s behavior is predictable and safe at compile time.

#### Triage

The question asks how to change an organization’s approach so that correctness in concurrency is enforced by design rather than relying on reviewer vigilance. This means moving from dynamic language practices like Java with "thread-safe" comments to a more rigorous, statically typed system such as Rust. We need to address the root cause of the race condition: improper checks-then-act logic.

#### First-Principles Solution
Understand the Problem: The issue lies in the current practice where developers use "thread-safe" comments but implement checks-then-act logic that can still fail due to the nature of concurrent execution.
Transition to Static Typing and Strong Concurrency Guarantees:
Rust: Rust’s ownership model ensures memory safety and thread safety at compile time, reducing the likelihood of race conditions.
Language Choice: Select a language that enforces stricter concurrency rules, such as Rust, Swift, or Go with its channel-based communication approach.
API Design:
Immutable Data Structures: Prefer immutable data structures to ensure they are inherently thread-safe.
Concurrency Models: Use actor models or message-passing paradigms where possible.
Code Review and Testing:
Automated Tests: Implement comprehensive unit and integration tests that cover all concurrency paths.
Static Analysis Tools: Use tools like Clang-Tidy, Rust’s clippy, or Go’s staticcheck to catch potential issues early.
Documentation and Training:
Guidelines: Develop clear coding guidelines that enforce the use of strong types and concurrency-safe practices.
Training Programs: Provide training for developers on modern concurrency techniques and best practices.

#### Closing Summary

By shifting from dynamic languages like Java to statically typed languages such as Rust, we can enforce correctness in concurrency through design principles. This approach ensures that race conditions are caught at compile time, reducing the burden on reviewers and improving the overall quality of the codebase.

#### Common Mistakes/Follow-Ups
Overlooking Automated Testing: Failing to implement comprehensive automated tests can lead to undetected issues.
Ignoring Tooling: Not using static analysis tools can result in missed concurrency-related errors.
Misaligned Team Skills: Transitioning to a new language or paradigm requires training and may face resistance.

**Comparison:**

```
Language	Thread-Safety Mechanism	Compile-Time Checks
Rust	Ownership model	Yes
Java	Comments, synchronization	No
```

**Code:**

```
RUST
Copy
// Example Rust code snippet demonstrating immutable data structures
struct Wallet {
    balance: i32,
}

impl Wallet {
    fn debit(&mut self, amount: i32) -> Result<(), String> {
        if amount > 0 && self.balance >= amount {
            self.balance -= amount;
            Ok(())
        } else {
            Err("Insufficient funds".to_string())
        }
    }
}
```

**Example input / output:**

```text
Input:
- Wallet balance = 100
- Debit request amount = 50

Output:
- Result: Ok(), new balance = 50
```

**Data:**

```json
{
  "wallet": {
    "balance": 100
  },
  "debit_request": {
    "amount": 50
  }
}
```

**Flow:**

1. Immutable Data Structures
2. Rust's Ownership Model
3. Automated Tests
4. Automated Test Suite
5. Static Analysis Tools
6. Clang-Tidy, Rust clippy

