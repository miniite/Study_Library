---
type: "practice"
difficulty: "Deep"
---

# You must decide the org standard for how schema-registry compatibility levels (backward, forward, full, transitive) map to what a pipeline is allowed to deploy for event-driven services; set defaults that are safe yet not paralyzing, and explain what breaks when a team picks the wrong compatibility level.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Evolution in Event Driven Deploys](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Schema_Evolution_in_Event_Driven_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this DevOps Architect/Principal role, we need to decide how schema-registry compatibility levels map to deployment permissions in our CI/CD pipelines. The core challenge is balancing flexibility with safety to avoid breaking changes while ensuring teams can develop and deploy event-driven services efficiently. We'll start by understanding the key concepts, then lay out a strategy that sets sensible defaults and explains potential pitfalls.

#### Triage

The question revolves around defining compatibility levels (backward, forward, full, transitive) in our schema-registry and mapping them to CI/CD pipeline permissions. The goal is to ensure that deployments do not break existing services while allowing teams the freedom to evolve schemas over time. We need to consider edge cases like data loss or compatibility issues when choosing the wrong level.

#### First Principles Solution

Understand Compatibility Levels:

Backward Compatible: Only newer versions can be added; older consumers continue to use old schema.
Forward Compatible: Older versions can still be produced by new producers, ensuring backward compatibility but allowing changes that do not break existing consumers.
Full Compatibility: Both producers and consumers must fully adhere to the latest schema version.
Transitive Compatibility: Ensures that any schema change in a linked service is propagated correctly.

Define Defaults:

Pipeline Default: Set default compatibility to Forward for development pipelines, ensuring changes are safe but allowing some evolution.
Staging/Production Default: Use Full or Transitive for staging and production environments to maintain strict compliance with service contracts.

Team Choice:

Teams should be allowed to override the default settings based on their specific requirements and risk tolerance.
Provide clear guidelines and documentation to help teams make informed decisions.
Step-by-Step Solution

Schema Registry Setup:

Configure schema registry with appropriate compatibility levels as default values.
Ensure that each service has a well-defined set of schemas for incoming and outgoing messages.

CI/CD Pipeline Integration:

Integrate schema validation checks into the CI/CD pipeline to ensure compatibility before deployment.
Use tools like Confluent Schema Registry or Kafka's built-in schema compatibility checks to enforce these rules during the build process.

Risk Assessment:

Conduct a risk assessment for each service and its dependencies to determine appropriate compatibility levels.
Ensure that critical services have stricter controls, while non-critical services can use more flexible settings.

Documentation and Training:

Document the schema compatibility strategy in detail, including examples and best practices.
Provide training sessions for developers and DevOps engineers on how to handle schema changes and compatibility levels.

Monitoring and Alerts:

Set up monitoring alerts to notify teams when compatibility issues are detected during testing or production.
Implement automatic rollbacks if necessary to prevent service disruption.

#### Closing Summary

Setting the right compatibility level is crucial for maintaining a robust event-driven architecture. By default, we can set forward compatibility for development environments and full compatibility for staging/production to ensure reliability. Teams should be empowered to make informed decisions based on their specific needs but must adhere to strict guidelines to prevent disruptions.

#### Common Mistakes
and Follow-Ups
Not Enforcing Compatibility: Not integrating schema validation into the CI/CD pipeline can lead to undetected breaking changes.
Overly Strict Defaults: Using full compatibility in development environments may hinder productive development cycles.
Ignoring Dependencies: Failing to account for transitive dependencies can cause unexpected issues in linked services.

In summary, our approach ensures a balance between flexibility and safety by setting sensible defaults and providing clear guidelines. This strategy helps us maintain the integrity of our event-driven systems while enabling teams to innovate effectively.

**Comparison:**

```
Compatibility Level	Description
Backward	Only newer versions allowed, ensures existing consumers work.
Forward	Older versions can produce new schemas, allows some evolution.
Full	Strict schema adherence required for both producers and consumers.
Transitive	Ensures compatibility across linked services.
```

**Code:**

```python
# Example CI/CD pipeline step for schema validation
def validate_schema(version, is_backward_compatible):
    if not is_backward_compatible:
        # Perform forward compatibility check
        pass
    else:
        # Perform backward compatibility check
        pass

pipeline_steps = [
    {
        "step": "validate",
        "version": "1.0",
        "is_backward_compatible": True,
    },
    {
        "step": "deploy",
    }
]
```

**Example input / output:**

```text
Input:
- Service A is deployed with compatibility level 'forward'.
- Service B depends on Service A and uses 'transitive' compatibility.

Output:
- Service A can evolve its schema without breaking Service B.
- Any changes in Service A's schema will be propagated to Service B as long as they remain compatible.
```

**Data:**

```json
{
  "service_name": "example-service",
  "compatibility_level": "forward",
  "dependencies": [
    { "name": "dependent-service", "version": "1.2" },
    { "name": "another-service", "version": "0.9" }
  ]
}
```

**Flow:**

1. No
2. Yes
3. Schema Registry
4. Is Compatibility Level Set?
5. Default Forward
6. Check Specific Rules
7. Validate and Deploy

