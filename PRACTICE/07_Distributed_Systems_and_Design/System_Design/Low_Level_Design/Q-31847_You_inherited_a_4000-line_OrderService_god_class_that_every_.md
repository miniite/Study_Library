---
type: "practice"
difficulty: "Expert"
---

# You inherited a 4000-line OrderService god class that every team touches and is a merge-conflict magnet across the org; lay out a multi-quarter strangler-style decomposition into cohesive modules with no big-bang rewrite and no code freeze, and how you'd measure progress.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Design Pattern Governance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/Design_Pattern_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this complex scenario, we'll employ a strangler-style decomposition approach to break down the monolithic OrderService class into more manageable modules over multiple quarters without causing a big-bang rewrite or a code freeze. The goal is to isolate and replace sections of functionality incrementally while maintaining system stability.

First, let's triage the question: the interviewer wants us to outline a strategic plan for refactoring an existing service that touches many teams, ensuring minimal disruption and measuring progress effectively. We need to make assumptions about the current codebase structure, dependencies, and the expected timeline. For this example, we'll assume a typical e-commerce application.

Triaging the Problem
Current State: The OrderService class is 4000 lines long and every team interacts with it, leading to frequent merge conflicts.
Objective: Refactor the service into cohesive modules without causing a big-bang rewrite or disrupting ongoing development.

#### Cons
traints: Limited downtime for refactoring work.

#### First-Principles Solution
Step 1: Identify Functional Units

We'll start by identifying distinct functional units within OrderService. These could include order creation, payment processing, inventory management, and shipment tracking. Each unit will form a new module that can be developed and tested independently.

Order Service
Isolate Functional Units
Order Creation
Payment Processing
Inventory Management
Shipment Tracking
Step 2: Define Module Interfaces

Each new module will have clear, well-defined interfaces. We'll create a contract layer that acts as an intermediary between the existing service and new modules. This ensures that changes in one module do not affect others.

Existing OrderService
Contract Layer
Order Creation Module
Payment Processing Module
Inventory ManagementModule
Shipment Tracking Module
Step 3: Implement and Integrate

We'll implement each module incrementally, starting with simple functionality that can be tested independently. As we progress, we’ll integrate these modules back into the existing service, ensuring seamless interaction through the contract layer.

Existing OrderService
Implement Module C1
Contract Layer
Order Creation Module
Implement Module C2
Payment Processing Module
Implement Module C3
Inventory ManagementModule
Implement Module C4
Shipment Tracking Module
Step 4: Measure Progress

We’ll measure progress through several metrics:

Code Coverage: Ensure that each module is well-tested.
Integration Tests: Verify that modules work correctly together.
Performance Metrics: Monitor any performance impacts during the transition.
MARKDOWN
| Metric | Description |
|--------|-------------|
| Code Coverage | Percentage of code covered by tests. |
| Integration Tests | Number of passing integration tests. |
| Performance Impact | Comparison of current and expected system performance. |
Step 5: Continuous Improvement

After initial deployment, we’ll continuously gather feedback from teams using the service. This will help us identify any issues early and make necessary adjustments to improve the modules.

#### Closing Summary

By breaking down OrderService into smaller, independent modules, we can refactor the codebase incrementally while minimizing disruption. Using a contract layer ensures that changes in one module do not affect others. Measuring progress through key metrics will help us track success and identify areas for improvement.

#### Common mistakes
include:

Overlooking existing dependencies.
Not defining clear interfaces between modules.
Failing to measure performance impacts during the transition.

**Comparison:**

```
Metric	Description
Code Coverage	Percentage of code covered by tests.
Integration Tests	Number of passing integration tests.
Performance Impact	Comparison of current and expected system performance.
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[Existing OrderService] --> B{Isolate Functional Units}
    B --> C1(Order Creation)
    B --> C2(Payment Processing)
    B --> C3(Inventory Management)
    B --> C4(Shipment Tracking)
```

**Flow:**

1. Existing OrderService
2. Implement Module C1
3. Contract Layer
4. Order Creation Module
5. Implement Module C2
6. Payment Processing Module
7. Implement Module C3
8. Inventory ManagementModule
9. Implement Module C4
10. Shipment Tracking Module

