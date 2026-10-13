---
type: "practice"
difficulty: "Expert"
---

# Design a workflow-orchestration platform bet for the org: adopt Temporal/Cadence, build on top of a queue with your own saga framework, or use step functions. Frame the multi-year implications for durability guarantees, developer velocity, operational load, and lock-in, and where you'd draw the line.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Choosing an Event Backbone](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Choosing_an_Event_Backbone.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a workflow-orchestration platform for our organization, we need to weigh several critical factors including durability guarantees, developer velocity, operational load, and potential vendor lock-in. Let's break down each approach and consider their implications over the course of multiple years.

#### Triaging the Question

The question asks us to evaluate three different approaches: adopting Temporal/Cadence, building on top of a queue with our own saga framework, or using AWS Step Functions. The key aspects we need to consider are:

Durability Guarantees: How well the system handles failures and ensures that workflows complete even when parts of the system fail.
Developer Velocity: The ease with which developers can build, deploy, and maintain workflows within the platform.
Operational Load: The complexity and resources required for ongoing maintenance and scaling.
Lock-In: The level of vendor dependency that could arise if we choose a specific product.
Step-by-Step Solution
1. Temporal/Cadence

Temporal is an open-source workflow orchestration system, originally developed by Netflix. It uses the Saga pattern to handle complex workflows with high reliability and fault tolerance.

Durability Guarantees: Temporal provides robust durability guarantees through its retry mechanisms, compensating actions, and full transactional support. This ensures that even if a part of the workflow fails, it can be retried or compensated for until completion.

Developer Velocity: Temporal’s API is designed to be developer-friendly, making it relatively easy to write workflows in Go. However, setting up and configuring Temporal requires understanding its concepts like Activities and Workflows, which might have a learning curve.

Operational Load: Temporal adds significant operational complexity due to the need for ZooKeeper or etcd for consensus. This increases the operational burden as these systems require careful management and monitoring.

Lock-In: Using open-source tools like Temporal provides flexibility but also requires setting up and maintaining infrastructure, which can lead to vendor lock-in through custom integrations and configurations.

2. Custom Saga Framework on Queue

Building a saga framework on top of an existing queue system (like RabbitMQ or Kafka) allows us to tailor the solution specifically for our needs. This approach gives more control but requires significant development effort upfront.

Durability Guarantees: Custom implementations can provide high durability if properly architected, with retries and compensating actions being well-defined in code.

Developer Velocity: Developing a custom saga framework can be time-consuming due to the need for comprehensive error handling and compensations. However, once implemented, it offers flexibility and customization that might not be available out-of-the-box.

Operational Load: The operational load is lower than Temporal because we control every aspect of our queue system. However, this comes with increased maintenance efforts as custom solutions require more robust testing and monitoring.

Lock-In: Since the solution is built in-house, there's less risk of vendor lock-in. However, it requires maintaining a significant amount of custom code and infrastructure.

3. AWS Step Functions

AWS Step Functions provide an easy-to-use serverless workflow service that supports various triggers and activities, including HTTP and custom actions.

Durability Guarantees: Step Functions support complex workflows with retries and error handling, ensuring high reliability even in the face of failures.

Developer Velocity: Using Step Functions is straightforward, with a visual designer that allows rapid prototyping and deployment. However, developers need to understand AWS services and step functions.

Operational Load: AWS manages much of the operational complexity, reducing the ongoing burden for our team. However, it introduces some vendor lock-in as we depend on AWS services.

Lock-In: There is a moderate risk of vendor lock-in due to reliance on AWS Step Functions and other associated services. But the managed nature reduces some aspects of maintenance and infrastructure management.

#### Closing Summary

In summary, the choice between Temporal/Cadence, a custom saga framework, or AWS Step Functions depends heavily on our organizational needs and priorities. Temporal offers robust durability guarantees but requires significant operational effort. A custom saga framework provides flexibility but is complex to implement and maintain. AWS Step Functions strike a balance by offering ease of use with some vendor lock-in.

#### Common Mistakes/Follow-Ups
Mistake: Overestimating the simplicity of a custom solution without considering ongoing maintenance.

#### Follow-Up
Discussing specific failure scenarios and how each system handles them, e.g., network partitions or long-running tasks.
Mistake: Ignoring lock-in considerations for long-term projects.

#### Follow-Up
Evaluating alternatives to Step Functions that might provide similar features with less vendor dependency.

**Comparison:**

| Workflow Type | Temporal | Custom Saga | Step Functions |
|---------------|----------|-------------|----------------|
| Durability    | High     | High        | High           |
| Velocity      | Developer-friendly API | Time-consuming to implement and maintain | Easy setup but requires AWS understanding |
| Load          | Significant ZooKeeper/etcd management required | Custom maintenance efforts | Lower operational load due to managed services |
| Lock-In       | Open-source, but custom integrations | In-house solution with less risk of vendor lock-in | Moderate vendor lock-in with AWS Step Functions |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Temporal/Cadence] --> B{High Durability}
    B --> C[Custom Saga Framework on Queue]
    A --> D[Easy Setup but Complex Maintenance]
    C --> E[Moderate Operational Load, High Flexibility]
    D --> F[AWS Step Functions: Balanced Choice]
    E --> G[Low Vendor Lock-In, Customized Solution]
    F --> H[Managed Services with Moderate Lock-In]
```

**Example input / output:**

| Workflow | Temporal | Custom Saga | Step Functions |
|----------|----------|-------------|----------------|
| Task A   | High     | High        | High           |
| Task B   | Medium   | Medium      | Low            |
| Task C   | Low      | Low         | High           |

**Data:**

```json
{
  "temporal": {
    "durability": "High",
    "velocity": "Developer-friendly API",
    "load": "Significant ZooKeeper/etcd management required",
    "lock_in": "Open-source, but custom integrations"
  },
  "custom_saga": {
    "durability": "High with proper implementation",
    "velocity": "Time-consuming to implement and maintain",
    "load": "Custom maintenance efforts",
    "lock_in": "In-house solution with less risk of vendor lock-in"
  },
  "step_functions": {
    "durability": "High, AWS-managed services",
    "velocity": "Easy setup but requires AWS understanding",
    "load": "Lower operational load due to managed services",
    "lock_in": "Moderate vendor lock-in with AWS Step Functions"
  }
}
```

**Flow:**

1. Workflow Choices
2. Temporal/Cadence
3. High Durability
4. Custom Saga Framework onQueue
5. Easy Setup but ComplexMaintenance
6. Moderate Operational Load,High Flexibility
7. AWS Step Functions: BalancedChoice
8. Low Vendor Lock-In,Customized Solution
9. Managed Services withModerate Lock-In

