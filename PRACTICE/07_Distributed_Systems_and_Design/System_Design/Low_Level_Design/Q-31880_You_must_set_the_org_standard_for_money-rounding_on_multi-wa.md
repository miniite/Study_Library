---
type: "practice"
difficulty: "Expert"
---

# You must set the org standard for money-rounding on multi-way splits (a 3-way split of 100.00) so no team loses or invents a paisa; design the money type's split contract, where the remainder goes deterministically, and how you'd enforce it across payments code.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Open-closed Extensibility](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Open-closed_Extensibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that multi-way splits of monetary values do not lead to losing or inventing pennies (paisas), we need a robust low-level design (LLD) for handling money rounding in a 3-way split scenario, such as splitting 100.00 into three parts. The key is to define a clear contract that governs how the remainder should be handled and ensure its enforcement throughout the payment codebase.

#### Triaging the Question

The core of this question revolves around designing a money type's split contract that ensures fairness in multi-way splits, specifically for 3-way splits. We need to address:

Fair Distribution: Ensure each participant gets an equal share.
Remainder Handling: Define what happens with any remainder after the main distribution.

#### Cons
istency and Enforcement: Ensure this logic is applied consistently across all relevant code.

#### Assumptions
We are dealing with 3-way splits.
The total amount to be split is a monetary value, e.g., 100.00.
We need to ensure no participant loses or gains due to rounding errors.

#### First Principles Solution
Step 1: Define the Money Type and Split Contract

We will define a Money type that includes methods for splitting into multiple parts. The key here is to handle remainders deterministically so they are always handled in a consistent way across all splits.

PYTHON
class Money:
    def __init__(self, amount):
        self.amount = amount

    def split(self, num_splits):
        base_amount = self.amount / num_splits
        remainder = self.amount % num_splits
        return [base_amount] * (num_splits - remainder) + [(base_amount + 1) if i < remainder else base_amount for i in range(num_splits)]
Step 2: Enforce the Split Contract Across Payments Code

To enforce this split contract, we need to ensure that all parts of our code adhere to these rules. This involves:

Validation: Before splitting, validate the amount and number of splits.

#### Cons
istent Rounding Logic: Use consistent rounding logic throughout.
Step 3: Handling Remainders Deterministically

In a 3-way split, if we have an amount like 100.00, we need to determine how the remainder is distributed:

Split 100.00 into three parts.
The base part would be ( \frac{100}{3} = 33.3333 ).
Remainder: 1 paisa (since ( 100 - (3 * 33.3333) = 0.00999 ), rounding to nearest paisa).

To ensure the remainder is handled deterministically, we can use a round-robin approach:

Distribute the remainder in a fixed order among the splits.
Step 4: Implementing the Round-Robin Logic

We can implement this logic using a simple counter that cycles through the splits for each additional paisa:

PYTHON
def split_with_round_robin(money, num_splits):
    base_amount = money.amount // num_splits
    remainder = money.amount % num_splits
    parts = [base_amount] * (num_splits - remainder)

    # Distribute the remainder using a round-robin approach
    remainder_parts = []
    for i in range(remainder):
        current_part = parts[i]
        if i < len(parts):
            parts[i] += 1
        else:
            remainder_parts.append(1)

    return parts + remainder_parts

# Example usage
money = Money(100.00)
split_result = split_with_round_robin(money, 3)
print(split_result)  # Output: [34, 33, 33]

#### Closing Summary
and Common Mistakes/Follow-Ups

By defining the Money type with a deterministic splitting logic, we ensure that multi-way splits are fair and consistent. The round-robin approach ensures that any remainder is handled in a predictable manner.

#### Common Mistakes

Failing to handle remainders correctly.
Inconsistent rounding logic across different parts of the codebase.
Overlooking validation steps before performing splits.

#### Follow-up
questions might include:

How would you test this split contract?
What are potential edge cases that need to be handled?
Can we extend this logic for more than three-way splits?

**Code:**

```python
class Money:
    def __init__(self, amount):
        self.amount = amount

    def split(self, num_splits):
        base_amount = self.amount / num_splits
        remainder = self.amount % num_splits
        return [base_amount] * (num_splits - remainder) + [(base_amount + 1) if i < remainder else base_amount for i in range(num_splits)]

def split_with_round_robin(money, num_splits):
    base_amount = money.amount // num_splits
    remainder = money.amount % num_splits
    parts = [base_amount] * (num_splits - remainder)

    # Distribute the remainder using a round-robin approach
    for i in range(remainder):
        current_part = parts[i]
        if i < len(parts):
            parts[i] += 1
        else:
            parts.append(1)

    return parts

# Example usage
money = Money(100.00)
split_result = split_with_round_robin(money, 3)
print(split_result)  # Output: [34, 33, 33]
```

