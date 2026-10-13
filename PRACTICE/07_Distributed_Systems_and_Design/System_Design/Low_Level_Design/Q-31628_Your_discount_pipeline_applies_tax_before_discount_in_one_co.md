---
type: "practice"
difficulty: "Deep"
---

# Your discount pipeline applies tax before discount in one code path and after in another, causing rupee mismatches customers complain about; how do you enforce a single canonical ordering in the design?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rule Engines and Pricing](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Rule_Engines_and_Pricing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

The key issue is ensuring consistent tax application order across your pipeline to avoid rupee mismatches. This requires rethinking the data flow and introducing canonical interfaces to enforce a single, well-defined sequence.

#### Triaging the Question

This question probes deep understanding of low-level design principles, specifically around enforcing consistency in a complex system. The interviewer wants to assess your ability to identify root causes, propose robust solutions, and ensure that all parts of the system follow a clear, unambiguous process.

#### Assumptions

The current pipeline processes transactions with two paths: one where tax is applied before discount, and another after.
The issue lies in these differing tax-application orders leading to discrepancies.
Customers are complaining about rupee mismatches due to inconsistent tax application.
Step-by-Step Solution From First Principles

Identify the Problem Scope:

Both paths should ideally process transactions in a consistent manner, i.e., either always applying tax before discount or vice versa.

Isolate Key Components:

Transaction Processing (TP) Module
Tax Calculation Module
Discount Application Module

Design Canonical Interfaces:

Define clear interfaces for each module that dictate the order of operations.
Use a standardized API for tax calculation and discount application.

Implement a Centralized Control Mechanism:

Introduce an orchestration layer to manage the flow of transactions.
Ensure this layer enforces the canonical sequence.

Modularize and Decouple Components:

Keep each module independent but interdependent via well-defined interfaces.
Avoid tightly coupling modules directly; instead, use dependency injection or other design patterns.

Ensure Consistency Across the Pipeline:

Apply a consistent policy across all transaction processing paths.
Use configuration or flags to allow dynamic switching between tax-first and discount-first flows but ensure only one path is active at any time.
Detailed Solution
Step 1: Define Canonical Interfaces
Transaction
Orchestrator
Tax Calculator
Discount Applier
Final Output
Transaction: Represents the input transaction data.
Orchestrator: Manages the flow and ensures consistent processing order.
Tax Calculator: Calculates tax based on the given policy (before or after discount).
Discount Applier: Applies discounts to the transaction.
Step 2: Implement Orchestration Layer

The orchestrator is responsible for ensuring that transactions always follow a canonical path. Here’s an example of how it might look in pseudocode:

PYTHON
class Orchestrator:
    def process_transaction(self, transaction):
        if self.use_tax_before_discount():
            # Apply tax first
            transaction = self.tax_calculator.calculate_tax(transaction)
            # Apply discount second
            transaction = self.discount_applier.apply_discount(transaction)
        else:
            # Apply discount first
            transaction = self.discount_applier.apply_discount(transaction)
            # Apply tax second
            transaction = self.tax_calculator.calculate_tax(transaction)
        return transaction

    def use_tax_before_discount(self):
        # Logic to determine if we should apply tax before discount
        return True  # This can be configurable based on business rules
Step 3: Ensure Consistency Across the Pipeline

Ensure that both paths in your codebase follow this canonical sequence. Any deviation should trigger a review and update.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary
By introducing a centralized orchestrator, defining clear interfaces, and enforcing consistent processing order, we ensure that the pipeline processes transactions uniformly, thereby avoiding rupee mismatches. This approach also provides flexibility to switch between different tax-discount policies without breaking existing components.

#### Common Mistakes

Failing to recognize the need for an orchestration layer.
Overcomplicating individual modules without recognizing their roles in a larger system.
Not defining clear interfaces, leading to tightly coupled code that is harder to maintain and test.

#### Follow-Ups

How would you handle edge cases where tax or discount calculations fail?
What are the performance implications of introducing an orchestrator?
Can you discuss how this solution scales with increased transaction volume?

**Comparison:**

```
Amount	Tax Rate	Discount Rate	Before Tax, After Discount (True)	Before Discount, After Tax (False)
100	10	20	88	88
```

**Code:**

```python
# Pseudocode for Orchestrator
class Orchestrator:
    def process_transaction(self, transaction):
        if self.use_tax_before_discount():
            # Apply tax first
            transaction = self.tax_calculator.calculate_tax(transaction)
            # Apply discount second
            transaction = self.discount_applier.apply_discount(transaction)
        else:
            # Apply discount first
            transaction = self.discount_applier.apply_discount(transaction)
            # Apply tax second
            transaction = self.tax_calculator.calculate_tax(transaction)
        return transaction

    def use_tax_before_discount(self):
        # Logic to determine if we should apply tax before discount
        return True  # This can be configurable based on business rules
```

**Example input / output:**

```text
- **Input**: `{ "amount": 100, "tax_rate": 10, "discount_rate": 20 }` with `use_tax_before_discount = True`
- **Output**:
  - After Tax Calculation: { "amount": 110 }
  - After Discount Application: { "final_amount": 88 }

- **Input**: `{ "amount": 100, "tax_rate": 10, "discount_rate": 20 }` with `use_tax_before_discount = False`
- **Output**:
  - After Discount Application: { "amount": 80 }
  - After Tax Calculation: { "final_amount": 88 }
```

**Data:**

```json
{
  "transaction": {
    "amount": 100,
    "tax_rate": 10,
    "discount_rate": 20
  }
}
```

**Flow:**

1. Transaction
2. Orchestrator
3. Tax Calculator
4. Discount Applier
5. Final Output

