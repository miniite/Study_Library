---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd version and deploy a gRPC service so that adding a field, deprecating a field, and removing a field each map to distinct, safe pipeline stages across producer and consumers.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Shared Pipeline Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure robust, controlled, and backward-compatible changes in a gRPC service, we need to implement a versioning strategy that aligns with CI/CD pipelines. This involves separating the processes of adding, deprecating, and removing fields into distinct pipeline stages for both producers (servers) and consumers (clients). Here’s how you can achieve this:

TL;DR
Version Control: Use semantic versioning to manage API changes.
Deprecation Stages: Gradually introduce deprecation warnings and ensure backward compatibility during transitions.
Removal Stages: Plan a phased removal, allowing time for consumers to migrate off the deprecated fields.

#### Triaging the Question

The question is essentially asking how to manage API versioning and field changes in a gRPC service within a CI/CD pipeline. The core goal is ensuring that:

Adding new fields doesn’t break existing clients.
Deprecating fields provides time for updates without immediate failure.
Removing fields can happen safely once all consumers have been updated.

#### Assumptions
We use semantic versioning (X.Y.Z), where X is the major version, Y is minor, and Z is patch.
Major versions indicate breaking changes.
Minor versions represent backward-compatible additions or changes.
Patch versions are for bug fixes that do not affect compatibility.

#### First-Principles Solution
1. Version Control with Semantic Versioning
Use semantic versioning to manage API contracts. Each change, whether adding, deprecating, or removing fields, should result in a new version of the gRPC service.
For example:
PLAINTEXT
v1.0.0: Initial release
v2.0.0: Major changes (breaking)
v2.1.0: Minor changes (compatible with 2.x)
2. Deprecation Stages
Introduce a deprecation stage where deprecated fields are marked as such in the service definition.
Emit warnings during runtime when these fields are accessed by consumers.
Gradually increase the severity of the warning until it becomes an error, giving time for clients to migrate.
3. Removal Stages
Create a phased removal process where the field is marked as deprecated in the latest version and removed in subsequent versions.
Provide ample notice (typically one major version) before completely removing the field.
Detailed Steps

Add Field (Non-Breaking Change)

Add new fields to the service definition.
Increment the minor or patch version depending on whether it’s a breaking change or not.
Use protoc to generate updated client and server code.
Stage the changes in a feature branch.
CI/CD pipelines should run tests and ensure compatibility with older clients.

Deprecate Field (Breaking Change)

Mark fields as deprecated in the service definition.
Emit deprecation warnings during runtime for these fields.
Increment the minor version to indicate a breaking change.
Stage this in a separate branch.
Run tests and ensure that existing clients will continue to function without errors, but they should log deprecation warnings.

Remove Field (Non-Breaking Change)

After sufficient time has passed with a deprecation warning stage, remove the deprecated fields from the service definition.
Increment the minor version again if necessary to reflect this breaking change.
Stage and merge into main branches carefully.
Ensure thorough testing across all environments.

#### Closing Summary

By using semantic versioning, deprecation stages, and phased removals, we can ensure that adding new fields, deprecating existing ones, and removing them are handled in a controlled manner. This approach minimizes disruptions to users while allowing the service to evolve safely over time.

#### Common Mistakes/Follow-Ups
Failing to Communicate Changes: Ensure all stakeholders are informed about version changes.
Insufficient Testing: Rigorously test new versions before deploying them to production.
Overlooking Deprecation Warnings: Ensure deprecation warnings are properly handled and logged in consumers.

**Comparison:**

```
Stage	Version	Action
Add	v2.1.0 (minor)	Add new_field3
Deprecate	v2.2.0 (major)	Mark deprecated_field2
Remove	v2.3.0 (minor)	Remove deprecated_field2
```

**Code:**

```
PROTOBUF
Copy
syntax = "proto3";

// Example proto for a gRPC service with versioning

service MyService {
  rpc ExampleMethod(ExampleRequest) returns (ExampleResponse);
}

message ExampleRequest {
  string field1 = 1;
  int64 deprecated_field2 = 2 [deprecated = true]; // Deprecation stage
  int32 new_field3 = 3; // Add field stage
}
```

**Example input / output:**

```

Input:

PLAINTEXT
Copy
ExampleRequest { field1: "test", deprecated_field2: 5, new_field3: 8 }

Output:

PLAINTEXT
Copy
{
  "exampleResponse": {
    "field1": "value",
    "deprecatedFieldWarning": "Field 'deprecated_field2' is deprecated and may be removed in future versions.",
    "newFieldValue": 42
  }
}
```

**Data:**

```json
{
  "field1": "value",
  "deprecated_field2": 0,
  "new_field3": 42
}
```

**Flow:**

1. Service Change
2. Add Field
3. v2.1.0
4. Generate Code
5. Tests Passed
6. Merge to Main
7. Deprecate Field
8. v2.2.0
9. Emit Warnings
10. Tests Passed
11. Merge to Deprecation Branch
12. Remove Field
13. v2.3.0
14. Remove Code
15. Tests Passed
16. Merge to Main

