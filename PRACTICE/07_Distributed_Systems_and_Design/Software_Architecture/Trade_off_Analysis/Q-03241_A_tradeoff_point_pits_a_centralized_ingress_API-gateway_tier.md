---
type: "practice"
difficulty: "Deep"
---

# A tradeoff point pits a centralized ingress/API-gateway tier (uniform auth, rate-limit, observability) against per-team ingress ownership (autonomy, no shared bottleneck). How do you scenario-test this against a fleet-wide outage scenario and a team-velocity scenario?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Quality Attribute Trade-off Scenarios](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Quality_Attribute_Trade-off_Scenarios.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To analyze the tradeoff between a centralized ingress/API-gateway tier versus per-team ingress ownership in our architecture, we need to consider multiple dimensions, including fleet-wide outages, team velocity, and overall system resilience. Let's break down the problem step-by-step.

#### Triage
of the Question

The question is essentially asking us to compare two architectural designs:

A centralized ingress/API-gateway tier that offers uniform authentication, rate limiting, and observability.
A decentralized design where each team owns its own ingress, providing greater autonomy but potentially leading to a lack of shared resources.

The key is to understand the tradeoffs between these two approaches in terms of resilience against fleet-wide outages and support for team velocity.

#### Assumptions
The centralized approach provides uniformity and shared infrastructure benefits.
The decentralized approach promotes autonomous development cycles and reduces bottlenecks but may introduce complexity in managing different security policies.
We need to consider the impact on observability, reliability, and maintenance overhead.

#### First Principles Solution
Fleet-Wide Outage Scenario

Centralized Ingress:

#### Pros
A single point of failure is easier to manage; if it goes down, all teams are affected but only one team needs to fix the issue.

#### Cons
If the centralized gateway fails, every service and its dependent services will go down, leading to a cascading effect.

Decentralized Ingress:

#### Pros
Each microservice has its own ingress, which means if one fails, it only impacts that specific team’s applications.

#### Cons
Managing individual gateways can be complex; each gateway might have different security policies and rate limits, leading to inconsistent experiences for users.
Team Velocity Scenario

Centralized Ingress:

#### Pros
Teams benefit from shared infrastructure and policies, which speeds up deployment cycles as they don't need to manage their own gateways.

#### Cons
Changes in policy or upgrades can introduce delays since they affect all teams.

Decentralized Ingress:

#### Pros
Each team has full control over its ingress, allowing for rapid iteration and experimentation without affecting other teams.

#### Cons
There is no shared mechanism for global policies like rate limiting, which could lead to inconsistent user experiences if not managed properly.
Scenario Testing
Outage Scenario
Centralized Ingress:
TeamB
Failure
(TeamB Service)
Down
TeamA
Failure
(TeamA Service)
Down
Central Gateway
Fleet-wide outage
Decentralized Ingress:
Failure
Failure
TeamA Gateway
Down
TeamB Gateway
Down
Team Velocity Scenario
Centralized Ingress:
CentralIngress
"TeamA"
"DevOps"
CentralIngress
"TeamA"
"DevOps"
Update Policy
Apply Policy
Decentralized Ingress:
"IngressB"
"IngressA"
"TeamB"
"TeamA"
"DevOps"
"IngressB"
"IngressA"
"TeamB"
"TeamA"
"DevOps"
Update Policy
No Change

#### Closing Summary
and Common Mistakes/Follow-Ups

Summary: The centralized approach offers uniformity and resilience against fleet-wide outages but may slow down team velocity due to shared bottlenecks. The decentralized approach promotes autonomy and rapid development cycles but can lead to inconsistent experiences across teams.

#### Common Mistakes

Overlooking the importance of a resilient centralized gateway in high-failure scenarios.
Ignoring the complexity introduced by individual gateways, especially when it comes to maintaining consistent policies.

#### Follow-Ups

How does the size and maturity of the organization impact these tradeoffs?
Are there hybrid solutions that could mitigate some of the downsides while leveraging the benefits of both approaches?

By thoroughly evaluating these scenarios, we can make informed decisions based on our specific organizational needs and constraints.

**Code:**

```

flowchart TD; A[Central Gateway] --> B1(Fleet-wide outage) subgraph TeamA C[(TeamA Service)] -- Failure --> D1(Down) end subgraph TeamB E[(TeamB Service)] -- Failure --> F1(Down) end

CODE
Copy

```mermaid
flowchart TD;
G[TeamA Gateway] -- Failure --> H1(Down)
I[TeamB Gateway] -- Failure --> J1(Down)
CentralIngress
"TeamA"
"DevOps"
CentralIngress
"TeamA"
"DevOps"
Update Policy
Apply Policy
"IngressB"
"IngressA"
"TeamB"
"TeamA"
"DevOps"
"IngressB"
"IngressA"
"TeamB"
"TeamA"
"DevOps"
Update Policy
No Change
```

