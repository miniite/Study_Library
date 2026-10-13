---
type: "practice"
difficulty: "Deep"
---

# An LLD you shipped uses inheritance for AccountType and now Premium accounts need behaviour that violates the base contract; how do you spot this LSP break in review and refactor to composition safely?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Composition vs Inheritance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Composition_vs_Inheritance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the issue of violating the Liskov Substitution Principle (LSP) through inheritance in an LLD, we first need a clear understanding of the problem at hand. This involves recognizing that Premium accounts should not inherit directly from AccountType if it leads to behavior that breaks the contract defined by AccountType. Instead, we must refactor our design using composition to ensure that such violations are avoided.

The question is probing your ability to:

Understand and apply LSP: Confirming you grasp the importance of ensuring derived classes can replace base classes without breaking functionality.
Identify and fix inheritance issues in code reviews: Ensuring you can spot problematic behavior during code reviews, particularly when dealing with complex class hierarchies.
Refactor using composition over inheritance: Demonstrating your skill in refactoring designs to avoid inheritance pitfalls while maintaining the integrity of the system.

We start by assuming that AccountType has a base contract or interface that all account types should adhere to. Let’s consider an example where PremiumAccount needs additional behavior, such as premium features, but this new behavior violates the contract defined in AccountType.

Step-by-Step Solution
Identify Violations: First, review the codebase and test cases for PremiumAccount. Identify any operations or behaviors that are inconsistent with the base contract defined by AccountType.
Extract Interfaces/Contracts: Clearly define what the AccountType interface should be. For instance, if AccountType has a method calculateFee(), ensure this is strictly followed.
Refactor Using Composition: Instead of inheriting from AccountType, use composition to include an AccountType object within PremiumAccount. This allows you to add premium-specific behaviors without violating the base contract.

Here’s a sample refactoring approach:

JAVA
// Original Inheritance Approach (Violating LSP)
class PremiumAccount extends AccountType {
    public void applyDiscount() {
        // Violates LSP - modifies behavior not in AccountType
    }
}

// Refactored Composition Approach
class PremiumAccount {
    private final AccountType baseAccount;

    public PremiumAccount(AccountType baseAccount) {
        this.baseAccount = baseAccount;
    }

    public void applyPremiumFeatures() {
        // Add premium-specific features here
    }

    public void calculateFee() { // Ensures LSP is maintained
        baseAccount.calculateFee();
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

In this refactoring, the PremiumAccount class now uses composition to maintain the integrity of its base contract while allowing for additional premium-specific behaviors. This approach ensures that any subclass can be substituted for the base type without causing runtime errors or unexpected behavior.

#### Common mistakes
include:

Failing to fully understand and define the contracts/interfaces.
Overlooking subtle violations during code reviews.
Inadequate test coverage after refactoring, leading to bugs in production.

#### Follow-ups
might involve discussing additional design patterns like decorator or strategy for more complex scenarios.

**Code:**

```java
// Original Inheritance Approach (Violating LSP)
class PremiumAccount extends AccountType {
    public void applyDiscount() {
        // Violates LSP - modifies behavior not in AccountType
    }
}

// Refactored Composition Approach
class PremiumAccount {
    private final AccountType baseAccount;

    public PremiumAccount(AccountType baseAccount) {
        this.baseAccount = baseAccount;
    }

    public void applyPremiumFeatures() {
        // Add premium-specific features here
    }

    public void calculateFee() { // Ensures LSP is maintained
        baseAccount.calculateFee();
    }
}
```

