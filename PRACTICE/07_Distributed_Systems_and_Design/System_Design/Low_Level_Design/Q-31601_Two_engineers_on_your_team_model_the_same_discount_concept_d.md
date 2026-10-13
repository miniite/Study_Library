---
type: "practice"
difficulty: "Intermediate"
---

# Two engineers on your team model the same 'discount' concept differently in two services and it keeps causing bugs; how do you drive convergence on a shared domain model in a code review and what trade-offs do you weigh?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rule Engines and Pricing](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Rule_Engines_and_Pricing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we have two engineers modeling 'discount' differently within their services, leading to bugs when these services interact. The goal is to converge on a shared domain model through effective code review and balance trade-offs that might arise.

#### Triage

The key issue here is ensuring consistency in the discount concept across different parts of the application. When engineers work independently, they often implement similar concepts with subtle but critical differences, leading to potential bugs or even security vulnerabilities. The interviewer is probing your ability to:

Identify and document inconsistencies.
Facilitate collaboration between team members.
Balance trade-offs in design choices.

#### Assumptions
Both engineers are experienced and competent.
There is a shared understanding of the discount concept's importance.
The systems are not monolithic, but rather microservices or components that interact via APIs.
Time constraints for resolving this issue exist.

#### First Principles Solution

Document Current Implementations: Start by thoroughly documenting both current implementations to understand the differences and their implications.

Identify Core Requirements: List out the core requirements of a discount system, such as applying discounts in real-time, historical tracking, or batch processing.

Choose a Common Model: Based on the requirements, select one model that is most appropriate. For instance, if real-time discount application is crucial, choose an in-memory approach over a database-only solution to ensure low-latency performance.

Code Review and Collaboration:

Conduct detailed code reviews with both engineers.
Identify areas of divergence and discuss the trade-offs between each implementation.
Propose a unified model that balances the needs of both services.

Implement and Test: Once a consensus is reached, implement the chosen model in one service first as a prototype. After successful testing, migrate it to the other service.

Iterate if Necessary: If issues arise during integration, revisit the design with all stakeholders involved.

#### Closing Summary

By following these steps, you can drive convergence on a shared domain model effectively while considering various trade-offs. The key is clear documentation and active collaboration to ensure that both engineers understand the rationale behind the chosen solution.

#### Common Mistakes
and Follow-Ups
Ignoring Trade-Offs: Not weighing the pros and cons of each approach.
Lack of Clear Communication: Poor communication between engineers can lead to misunderstandings.
Ignoring Real-Time Requirements: Focusing only on storage mechanisms without considering real-time performance.

Common follow-ups might include:

What if both models are equally valid?
How do you handle edge cases, such as overlapping discounts?

**Comparison:**

```
Discount Type	Value	Start Date	End Date
Percentage	10%	2023-04-01T00:00:00Z	2023-06-30T23:59:59Z
Flat	$2.00	2023-04-01T00:00:00Z	2023-07-31T23:59:59Z
```

**Code:**

```python
# Example of a simple in-memory discount model
class DiscountModel:
    def apply_discount(self, product_id, quantity):
        # Simple logic for applying discounts
        pass

# Example implementation in Service A
def service_a_discount_logic(product_id, quantity):
    discount_model = DiscountModel()
    return discount_model.apply_discount(product_id, quantity)

# Example implementation in Service B (potentially different)
def service_b_discount_logic(product_id, quantity):
    # Different logic or data structure
    pass
```

**Example input / output:**

```

Input:

PYTHON
Copy
product_id = 1234
quantity = 5

Output for service_a_discount_logic:

PYTHON
Copy
discounted_price = $8.00 # Assuming original price was $10 and discount is 20%

Output for service_b_discount_logic (different implementation):

PYTHON
Copy
discounted_price = $7.95 # Different discount logic or value
```

**Data:**

```json
{
  "discount": {
    "type": "percentage",
    "value": 10,
    "start_date": "2023-04-01T00:00:00Z",
    "end_date": "2023-06-30T23:59:59Z"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Identify DivergentImplementations
6. Review and DocumentDifferences
7. Are Requirements Clear?
8. Choose a Common Model
9. Can We Integrate BothModels?
10. Integrate Both Models
11. Propose a Unified Model withTrade-offs

