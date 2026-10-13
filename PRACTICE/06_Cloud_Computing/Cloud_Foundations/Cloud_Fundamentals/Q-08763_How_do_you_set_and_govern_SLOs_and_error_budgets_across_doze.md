---
type: "practice"
difficulty: "Expert"
---

# How do you set and govern SLOs and error budgets across dozens of interdependent services so that one team's budget burn actually constrains its dependents, without creating a bureaucratic nightmare?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ SLOs and Error Budgets](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Availability_Zone_Resilience/SLOs_and_Error_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure effective Service Level Objectives (SLOs) and error budgets are set and governed across dozens of interdependent services in a cloud environment, we need to establish a balanced approach that promotes accountability while avoiding excessive bureaucracy. The core idea is to leverage a layered governance model that starts with clear definitions, aligns teams around shared goals, and uses automated tools for enforcement.

The question primarily probes how you can manage SLOs and error budgets effectively in a large-scale system where services are interdependent. This involves setting up a framework for defining, monitoring, and enforcing SLOs and error budgets, ensuring that the constraints imposed by one service's budget do not disproportionately burden its dependents or create bottlenecks.

#### Triaging the Question
SLOs and Error Budgets: These are key metrics used to define expected performance levels and acceptable failure rates for services.
Interdependent Services: In a complex system, one service's behavior can significantly impact others. For instance, high error rates in a database could lead to timeouts or degraded response times for dependent APIs.
Governance Model: The challenge is to create a scalable and manageable governance structure that allows teams to operate autonomously while ensuring they adhere to necessary constraints.

#### First-Principles Solution

Define SLOs and Error Budgets:

SLOs: Establish clear, measurable objectives for each service's performance.
Error Budgets: Allocate the allowable number of errors or failures per unit time based on business impact and SLOs.

Impact Analysis:

Perform a dependency graph analysis to understand how services interact and affect one another.
Identify critical paths where failure in one service could have cascading effects.

Budget Propagation and Constraints:

Use a top-down approach starting from the most critical services down to less critical ones.
Ensure that any budget reallocation or constraint imposed by a higher-level service is communicated and understood by its dependents.
Implement automated tools to enforce these constraints, ensuring that when one service exceeds its error budget, it triggers alerts and actions in downstream services.

Governance Layers:

Operational Layer: Teams own the SLOs and error budgets for their services but follow centralized guidelines.
Managerial Layer: Higher-ups can override or adjust budgets if necessary to prevent systemic failures.
Automated Enforcement Layer: Tools automatically enforce budget constraints, ensuring compliance without manual intervention.

Feedback Loops:

Regularly review and adjust SLOs and error budgets based on performance data and business needs.
Use metrics like availability, latency, and error rates to continuously monitor service health.

#### Closing Summary

By adopting a layered governance model that includes clear definitions of SLOs and error budgets, impact analysis, budget propagation, automated enforcement, and regular feedback loops, you can effectively manage the constraints across dozens of interdependent services. This approach ensures that no single team's actions excessively burden its dependents while maintaining overall system resilience.

#### Common Mistakes
and Follow-Ups
Bureaucratic Overreach: Teams may feel micromanaged if too many constraints are imposed without proper communication.
Overly Complex Models: Creating overly intricate models for budget propagation can lead to confusion and implementation challenges.
Inadequate Feedback Mechanisms: Without regular monitoring and adjustment, SLOs and error budgets may become ineffective over time.

Common follow-up questions might include:

How do you handle edge cases where a critical service's failure disproportionately affects multiple dependents?
What tools or platforms would you recommend for automating budget enforcement?
Can you provide an example of how you've implemented this approach in a real-world scenario?

**Comparison:**

```
Service	Availability SLO	Error Budget
Database	99.9%	1000
Web API	N/A	500
```

**Code:**

```python
# Example code snippet to illustrate automated error budget enforcement

def enforce_error_budget(service_name, current_errors):
    # Check if the service has exceeded its allowed errors
    threshold = get_error_threshold(service_name)
    if current_errors > threshold:
        trigger_alert(service_name)
        propagate_budget_reduction(service_name)

def get_error_threshold(service_name):
    # Retrieve error budget threshold for a given service
    pass

def trigger_alert(service_name):
    # Send alert to relevant stakeholders
    pass

def propagate_budget_reduction(service_name):
    # Propagate the reduction in error budget to dependent services
    pass
```

**Example input / output:**

```text
Input: Service 'database' exceeds its allowed errors.
Output: Alert triggered, and the error budget for 'web_api' is reduced by 10% to ensure it remains within its constraints.
```

**Data:**

```yaml
# Example configuration for SLOs and Error Budgets
services:
  - name: database
    slos:
      availability: "99.9%"
    error_budget: 1000

  - name: web_api
    slos:
      response_time: "200ms"
    error_budget: 500

# Dependency graph
dependencies:
  - source_service: database
    target_services:
      - web_api
```

**Flow:**

1. Yes
2. No
3. Service Exceeds Error Budget
4. Is Service Critical?
5. Trigger Alert
6. Propagate Error BudgetReduction
7. Notify Stakeholders
8. Adjust Dependent Services'Error Budgets

