---
type: "practice"
difficulty: "Deep"
---

# You inherit a gateway where every team gets root-equivalent config access and one team's typo took down checkout; design the least-privilege config authorization model (who can touch which routes, blast-radius scoping, break-glass) without turning the platform team into an approval bottleneck.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gateway Idempotency and Org Standards](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Gateway_Idempotency_and_Org_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to rethink our API Gateway's configuration access control from a least-privilege perspective. The current situation where every team has root-equivalent config access is fundamentally flawed and needs to be addressed to minimize the risk of accidental or malicious changes that could take down critical services like checkout.

Triaging the Problem

The core issue here is ensuring that each team has only the necessary permissions to configure their specific routes while preventing cascading failures from a single misconfiguration. The goal is to implement a robust, flexible, and least-privilege authorization model without creating an approval bottleneck for the platform team.

#### Assumptions

We need to define clear roles and permissions.
Changes should be monitored in real-time.
We will use a configuration management system that enforces these rules.
The platform team must be able to intervene quickly but not micromanage daily operations.
Step-by-Step Solution
1. Define Roles and Permissions

We start by defining the roles required for API Gateway management:

Platform Admin: Full access to all configurations, responsible for setting up initial policies.
Team Lead (TL): Can manage routes and configurations within their scope but cannot affect other teams' services directly.
Configuration Approver: Review and approve critical changes before they are deployed.
2. Implement Scope-Based Access Control

Each team should have a defined scope of responsibility, which includes the routes and sub-resources they can configure or modify. This scope is enforced by:

Route Definitions: Each route should be associated with one or more teams that have access to it.
Configuration Rules: Use a declarative rule engine to enforce these scopes.
3. Real-Time Monitoring and Alerting

Real-time monitoring of configuration changes can help detect and prevent accidental outages:

Change Logs: Maintain logs of all configuration changes, who made them, and the time.
Alerts: Set up alerts for critical changes (e.g., breaking changes to a live service).
4. Break-Glass Mechanism

In case of an emergency where immediate action is needed but no one can make decisions due to the absence of platform team members, a break-glass mechanism should be in place:

Emergency Approval: A predefined set of high-level approvals that can be used in emergencies.
Automated Rollbacks: Predefined rollback procedures for common issues.
5. Monitoring and Feedback

Continuous monitoring and feedback mechanisms ensure the system is working as intended:

Usage Metrics: Collect metrics on API usage to identify potential bottlenecks.
User Training: Regularly train team members on best practices and reporting incidents early.
Detailed Implementation
Platform Admin
Define Roles
Scope-Based Access Control
Real-Time Monitoring
Break-Glass Mechanism
Monitoring and Feedback

#### Closing Summary

By implementing a role-based access control system with scope-based permissions, real-time monitoring, and a break-glass mechanism, we can ensure that the API Gateway remains resilient to misconfigurations while preventing the platform team from becoming an approval bottleneck. This approach balances security and operational efficiency.

#### Common Mistakes
and Follow-Ups
Overly Complex Rules: Defining too many rules can lead to confusion and maintenance issues.
Lack of Monitoring: Not having real-time monitoring can result in delayed responses to critical issues.
Approval Bottlenecks: Ensuring that the break-glass mechanism is well-defined and tested to avoid delays.

**Comparison:**

| Role        | Permissions       |
|-------------|-------------------|
| Platform Admin | Full access to all configurations, initial setup of policies. |
| Team Lead   | Manage routes and configurations within their scope but cannot affect other teams' services directly. |
| Configuration Approver | Review and approve critical changes before deployment. |

**Code:**

```javascript
// Example configuration rule engine using JSON
{
  "rules": [
    {
      "team": "sales",
      "routes": ["/checkout"],
      "actions": ["read", "write"]
    },
    {
      "team": "engineering",
      "routes": ["/api/v1/*"],
      "actions": ["read", "write"]
    }
  ]
}
```

**Example input / output:**

| Action            | Input (Team, Route)           | Output (Access Allowed) |
|-------------------|-------------------------------|-------------------------|
| Team sales        | /checkout                     | Yes                     |
| Team engineering  | /api/v1/*                     | Yes                     |
| Team marketing    | /checkout                     | No                      |

**Data:**

```json
{
  "teams": [
    {"name": "sales", "role": "configuration"},
    {"name": "engineering", "role": "configuration"}
  ],
  "routes": [
    { "path": "/checkout", "team": "sales" },
    { "path": "/api/v1/*", "team": "engineering" }
  ]
}
```

**Flow:**

1. Team Member
2. Request Route Change
3. Configuration Rule Engine
4. Allowed?
5. Deny, Alert Platform Admin
6. Typos Fixed, Changes Applied

