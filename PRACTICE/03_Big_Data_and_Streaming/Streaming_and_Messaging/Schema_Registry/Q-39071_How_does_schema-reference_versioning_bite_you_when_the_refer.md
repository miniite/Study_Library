---
type: "practice"
difficulty: "Expert"
---

# How does schema-reference versioning bite you when the referenced (shared) schema evolves — what breaks and how do you version the parent?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Phased Schema Rollouts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Phased_Schema_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a shared schema in a Schema Registry evolves, issues can arise if the referencing schemas are not appropriately managed. This is particularly problematic when different services or applications rely on these shared schemas and evolve independently. The core challenge lies in ensuring that all dependent schemas are updated to accommodate changes in the parent schema without breaking downstream systems.

Let's break down this problem from first principles:

#### Triage

The question asks about the potential issues that arise due to versioning conflicts when a shared schema evolves, specifically focusing on how referencing schemas need to be managed and updated. The key points to consider are:

Shared Schema Evolution: How changes in the parent schema (shared across multiple services) affect the dependent schemas.
Versioning Strategy: Ensuring all referencing schemas are appropriately versioned to reflect their dependencies on the shared schema.

#### Assumptions
We have a Schema Registry that manages and version controls our schemas.
Multiple applications or services reference these schemas.
Schemas can evolve independently, sometimes requiring changes in the structure of the parent schema.
Step-by-Step Solution
1. Understanding Shared Schema Evolution
Suppose you have a shared schema called Person with fields like name, age, and address.
Over time, an application may need to add a new field email to this schema.
The updated Person schema is then stored in the Schema Registry.
2. Versioning Strategy for Referencing Schemas
Each referencing schema should be versioned independently to reflect its dependency on the shared schema.
For instance, if a service uses an older version of the Person schema (v1) and we evolve it to v2 with an additional field email, the service needs to be updated to use this new version.
3. Impact on Dependent Services
If a service continues to use the outdated schema, it will encounter issues when trying to process data that includes the new email field.
This could manifest as data validation failures or other runtime errors.
In some cases, services might crash if they cannot handle the new schema version.
4. Handling Schema Evolutions
When a shared schema evolves:
The Schema Registry should notify all dependent services about the change.
Services can choose to update their referencing schemas or continue using an older version for backward compatibility.
It's crucial to manage these transitions carefully, especially in production environments where downtime is critical.
5. Best Practices
Implement semantic versioning for shared and referenced schemas.
Use a change management process to ensure all stakeholders are aware of schema changes.
Provide fallback mechanisms or compatibility layers until services can be updated.

#### Closing Summary

Properly managing the evolution of shared schemas in a Schema Registry requires careful planning and execution. Ensuring that dependent schemas are versioned appropriately prevents breaking downstream systems when upstream schemas change. Best practices like semantic versioning, clear communication channels, and robust fallback strategies can help mitigate these issues.

#### Common Mistakes/Follow-Ups
Ignoring Versioning: Not updating referencing schemas can lead to data processing failures.
Lack of Communication: Failing to notify affected services about schema changes can result in unexpected downtime or bugs.
Insufficient Testing: Skipping thorough testing for compatibility between updated and old schema versions can introduce issues.

Would you like a deeper dive into any specific aspect, such as how to implement semantic versioning or best practices for change management?

**Comparison:**

```
ServiceB
SchemaRegistry
ServiceA
ServiceB
SchemaRegistry
ServiceA
Request schema version v1.0
Return schema v1.0
Process data using schema v1.0
Request updated schema version v2.0
Return updated schema v2.0 with additional field 'email'
Notify of schema update to v2.0
```

**Code:**

```python
# Example of defining a versioned schema in JSON Schema format
{
  "title": "Person",
  "type": "object",
  "properties": {
    "name": { "type": "string" },
    "age": { "type": "integer" }
  },
  "version": "1.0"
}

# Updated version with additional field
{
  "title": "Person",
  "type": "object",
  "properties": {
    "name": { "type": "string" },
    "age": { "type": "integer" },
    "email": { "type": "string" }
  },
  "version": "1.1"
}
```

**Example input / output:**

| Input (Old Schema) | Output (New Schema Validation Error) |
|--------------------|--------------------------------------|
| {"name": "John Doe", "age": 30} | ValidationError: Missing required field 'email' |

**Data:**

```json
{
  "person_v1": {
    "name": "John Doe",
    "age": 30
  },
  "person_v2": {
    "name": "Jane Smith",
    "age": 45,
    "email": "jane@example.com"
  }
}
```

**Flow:**

1. Shared Schema Evolves
2. Notify Dependent Services
3. Dependent Services Update toNew Version
4. Data Validation Passes
5. Data Validation Fails (due tomissing fields)
6. Service Crashes

