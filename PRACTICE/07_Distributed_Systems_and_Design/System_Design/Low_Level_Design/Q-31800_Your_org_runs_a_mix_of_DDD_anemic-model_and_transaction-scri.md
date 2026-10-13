---
type: "practice"
difficulty: "Expert"
---

# Your org runs a mix of DDD, anemic-model, and transaction-script styles across teams and onboarding engineers are constantly confused; do you standardize on one, and if so how do you sequence a multi-year convergence given the sunk cost in each style?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Design Pattern Governance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/Design_Pattern_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of inconsistent design patterns across teams and onboarding engineers being confused, we need to standardize our architectural approach. Given the current mix of Domain-Driven Design (DDD), anemic-model, and transaction-script styles, a multi-year convergence strategy is essential. This involves assessing the pros and cons of each style, aligning with organizational goals, and defining a clear path forward.

Firstly, let's understand what these design patterns mean in practice:

Domain-Driven Design (DDD): Focuses on modeling the business domain to create rich domain models that encapsulate business rules and logic.
Anemic Model: Keeps most of the logic outside the model classes, often leading to DTOs with minimal behavior.
Transaction Script: Ties together operations in a transactional script without much separation between data access and business logic.

Given our expertise as Backend Architects/Principles, we can make an informed decision by weighing the benefits and drawbacks of each approach:

#### Triaging the Question
The interviewer is probing for:
Our ability to evaluate different design patterns.
Strategies for managing organizational change.
Techniques for aligning technical choices with business goals.

#### Assumptions
Sunk Costs: Each team has invested time and effort into their preferred style, making it a significant sunk cost.
Organizational Goals: The overarching goal is to improve consistency, maintainability, and ease of onboarding for new engineers.
Long-term Vision: We need a multi-year plan that gradually standardizes the design patterns.
Step-by-Step Solution
Assess Current Practices:
Conduct a detailed audit of each team's current practices to understand the extent of DDD, anemic-model, and transaction-script usage.
Identify Key Stakeholders:
Engage with key stakeholders such as project managers, senior developers, and domain experts to gather insights.
Define Standards:
Propose a standard that leverages DDD principles while allowing for flexibility in implementation.
Phased Implementation:
Develop a phased approach: start with small-scale migrations to prove the concept, then scale up as teams demonstrate proficiency.
Detailed Phases
Pilot Projects (Year 1-2):
Select pilot projects that can benefit from DDD principles.
Train and mentor key team members on DDD practices.
Training and Documentation (Year 3-4):
Develop comprehensive training materials and documentation to support a smooth transition.
Organize workshops, webinars, and knowledge-sharing sessions.
Migrating Existing Systems (Year 5 onwards):
Gradually migrate existing systems while maintaining backward compatibility where necessary.
Leverage automation tools for consistent code quality checks.

#### Closing Summary

By adopting DDD as our standard design pattern, we can achieve a more maintainable and scalable architecture that aligns with business needs. The multi-year convergence plan ensures a smooth transition by leveraging pilot projects, training, and phased migrations.

#### Common Mistakes/Follow-Ups
Ignoring Sunk Costs: Failing to acknowledge the significant time and effort already invested may lead to resistance.
Rushing Adoption: Implementing DDD too quickly without proper preparation can cause friction among teams.
Lack of Training: Insufficient training or support for new patterns can result in poor adoption.

**Comparison:**

```
Phase	Duration	Activities
Pilot Projects	1-2 years	Select projects, train teams, implement DDD principles
Training and Documentation	3-4 years	Develop training materials, organize workshops, maintain documentation
Migrating Existing Systems	Ongoing	Gradual migration of systems, ensure backward compatibility
```

**Code:**

```python
# Example of a simple domain entity with behavior (DDD)
class Order:
    def __init__(self, id, customer_id):
        self.id = id
        self.customer_id = customer_id

    def place(self):
        # Business logic to place an order
        pass

# Anemic Model example
class OrderDTO:
    def __init__(self, id, customer_id):
        self.id = id
        self.customer_id = customer_id

# Transaction Script example
def process_order(order_id):
    # Logic to process the order in a single transaction
    pass
```

**Example input / output:**

```text
Input: Pilot projects selected for DDD adoption.
Output: Training materials and mentorship programs developed, workshops scheduled.

Request: Implementation of a new domain entity with behavior in an existing system.
Response: Domain model refactored to include business logic, tests added for validation.
```

**Data:**

```yaml
pilot_projects:
  - project_name: Order Fulfillment System
  - project_name: Customer Management Module

training_sessions:
  - session_date: "2023-10-15"
  - session_date: "2024-01-10"

migration_plan:
  - module: Order Processing
  - module: Inventory Management
```

**Flow:**

1. Initial Audit
2. Identify Stakeholders
3. Define Standards
4. Pilot Projects (1-2 years)
5. Training and Documentation(3-4 years)
6. Migrating Existing Systems(Ongoing)

