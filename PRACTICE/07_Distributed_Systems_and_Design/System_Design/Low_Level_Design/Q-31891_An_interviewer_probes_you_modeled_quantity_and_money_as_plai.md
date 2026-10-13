---
type: "practice"
difficulty: "Deep"
---

# An interviewer probes: you modeled 'quantity' and 'money' as plain integers everywhere and a multiply produced a nonsense unit (money times money); how would unit-typed value objects catch this at compile time, and what's the ergonomics cost you accept for that safety?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Domain Event Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/Domain_Event_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with financial calculations in software, it is crucial to ensure correctness and prevent logical errors like multiplying 'money' values by 'money' which results in a nonsensical unit like 'square money'. To address this issue at the compile time, we need to use units or value objects. This approach not only catches the type-related mistakes early but also enhances code readability and maintainability.

#### Triage
What's Being Probed

The interviewer is probing for an understanding of:

How to use unit-typed value objects in software design.
The trade-offs involved in adding compile-time safety mechanisms.
The impact on development ergonomics and productivity.

#### First-Principles Solution

To ensure that 'money' values are not mistakenly used as units, we can introduce a system where Money is treated as a distinct type with specific arithmetic operations defined only for valid combinations of types. This is commonly achieved through the use of value objects in object-oriented programming languages.

Step-by-Step Solution
Define Value Objects: Create Money, Quantity, and other relevant value objects to encapsulate data and ensure that these values are treated as distinct units.
Implement Type-Safe Arithmetic Operations: Define operations like addition, subtraction, multiplication, and division with appropriate return types. Ensure that multiplying two Money objects is not allowed without an explicit conversion or operation defined for it.
Compile-Time Safety: Leverage the language's type system to enforce compile-time safety where possible.

#### Example
Implementation in TypeScript
TYPESCRIPT
// Define a Money value object
class Money {
  private amount: number;

#### cons
tructor(amount: number) {
    this.amount = amount;
  }

  // Only allow multiplication with Quantity or other valid types
  multiply(quantity: Quantity): Money {
    return new Money(this.amount * quantity.value);
  }
}

// Define a Quantity value object
class Quantity {
  public value: number;

#### cons
tructor(value: number) {
    this.value = value;
  }
}
Handling Non-Sensical Operations

To catch nonsensical operations like Money * Money, we can use TypeScript's type guards and utility types to ensure that only valid operations are performed.

TYPESCRIPT
type ValidOperation<T> = T extends { multiply: (quantity: Quantity) => Money } ? T : never;

// Ensure that Money is treated as a valid operation target for multiplication with Quantity

#### cons
t validMoneyInstance: ValidOperation<Money> = new Money(100);

validMoneyInstance.multiply(new Quantity(5)); // Valid

// This will cause a TypeScript error because the type system enforces the correct usage.
// const invalidResult: ValidOperation<Money> = new Money(100).multiply(new Money(5));
Ergonomics and Cost Considerations

While this approach enhances safety, it comes with trade-offs:

Increased Development Time: Introducing value objects and ensuring type safety can add to the initial development time.
Code Complexity: Managing multiple types and operations can make the codebase more complex, requiring careful design and documentation.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Using unit-typed value objects in software design is a robust way to catch type-related errors at compile time. By leveraging TypeScript's strong typing system, we can enforce correct usage patterns and ensure that nonsensical operations like Money * Money are caught early. The key is to balance the added safety with development ergonomics.

#### Common mistakes
include:

Failing to define clear value objects and their arithmetic operations.
Overlooking type checks in operations, leading to runtime errors.
Ignoring the initial overhead of setting up such a system.

#### Follow-ups
could explore further optimizations or trade-offs specific to the project's needs.

**Code:**

```

class Money { private amount: number;

constructor(amount: number) { this.amount = amount; }

multiply(quantity: Quantity): Money { return new Money(this.amount * quantity.value); } }

class Quantity { public value: number;

constructor(value: number) { this.value = value; } }

CODE
Copy

```typescript
type ValidOperation<T> = T extends { multiply: (quantity: Quantity) => Money } ? T : never;
const validMoneyInstance: ValidOperation<Money> = new Money(100);
validMoneyInstance.multiply(new Quantity(5)); // Valid

// const invalidResult: ValidOperation<Money> = new Money(100).multiply(new Money(5));
```

