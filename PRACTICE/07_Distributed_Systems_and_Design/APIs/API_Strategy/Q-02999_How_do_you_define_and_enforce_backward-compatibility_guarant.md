---
type: "practice"
difficulty: "Expert"
---

# How do you define and enforce backward-compatibility guarantees for gRPC/protobuf APIs across a service mesh where producers and consumers deploy independently? What compatibility levels do you promise, and how do you catch a wire-breaking change before it ships?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Org-wide API Standards](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/Org-wide_API_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Backward-compatibility in gRPC/Protobuf APIs is crucial to ensure that services can evolve over time without breaking existing consumers. In a service mesh environment, where producers and consumers might be deployed independently, defining and enforcing these guarantees requires a strategic approach. This involves understanding the different levels of compatibility and implementing robust testing mechanisms.

Firstly, let's define what backward-compatibility means in this context. Backward-compatibility is typically divided into three levels: Semantic, Syntax (or API), and Behavioral. Semantic compatibility ensures that new features do not change existing contract semantics. Syntax compatibility involves changes to the wire format of messages without altering their meaning. Behavioral compatibility ensures that the functional behavior remains consistent.

To enforce these guarantees, we need a combination of design principles, tooling, and governance policies. Here’s how I would approach it:

Define Compatibility Levels:

Semantic: Ensure new versions do not alter existing semantics.
Syntax: Permit changes to the wire format as long as they are non-breaking (e.g., adding fields but marking them optional).
Behavioral: Prevent changes that could alter how the service behaves.

Implement Versioning Strategy:

Use semantic versioning for API versions, e.g., v1.0, v1.1.
Deprecate old APIs with a clear sunset plan, ensuring consumers have ample time to migrate.

Use Protobuf Features:

Optional Fields: Allow adding new optional fields without breaking backward compatibility.
Oneof: Use oneof groups to ensure only one field is required at a time if there’s a change in the message structure.

Leverage gRPC Tools:

gRPC-Gateway: For exposing APIs in a versioned manner and enabling graceful deprecation.
API Validation Tools: Use tools like protoc-gen-validate to enforce protocol buffer validation rules during compile-time.

Continuous Integration/Continuous Deployment (CI/CD):

Integrate compatibility checks into the CI pipeline, ensuring every change is tested for backward-compatibility.
Implement canary releases where new versions are deployed alongside old ones to catch any issues in a controlled environment.

Runtime Compatibility Checks:

Use runtime libraries that support checking protocol buffer messages against schema definitions at runtime.
Implement fallback mechanisms to handle deprecated fields or behaviors gracefully.

Governance and Documentation:

Document API changes thoroughly, providing clear guidelines on what is allowed and what is not.
Establish a governance board responsible for reviewing proposed changes before they are merged into the codebase.

By following these steps, we can ensure that any wire-breaking change is caught early in the development process. This approach balances innovation with stability, allowing services to evolve while maintaining compatibility with existing consumers.

In summary, defining and enforcing backward-compatibility guarantees involves a combination of robust design practices, tooling, governance policies, and continuous integration processes. The goal is to minimize disruption to consumers and maximize the flexibility of our service mesh architecture.

**Comparison:**

```
Compatibility Level	Description	Enforced by
Semantic	No change in existing semantics	Documentation, Governance
Syntax	Changes to wire format without meaning	Protobuf optional fields, versioning
Behavioral	No change in functional behavior	Runtime validation
```

**Code:**

```
PROTOBUF
Copy
syntax = "proto3";

package example;

option go_package = "example.com/example/api/v1;api_v1";

// Define a service with versioned messages
service ExampleService {
  rpc GetExample(GetRequest) returns (GetResponse);
}

message GetRequest {
  // Version 1.0: Optional field to be deprecated in the future.
  string optional_field = 1;
}

message GetResponse {
  string message = 1;
}
```

**Example input / output:**

```text
Input:
- New API version v2.0 released with a new optional_field that is marked as deprecated.

Output:
- Existing consumers using v1.0 continue to function without issues.
- New consumers can use the v2.0 schema but will ignore the deprecated field.
```

**Data:**

```yaml
# Example Protobuf Schema with Deprecation Annotations
package example.v2;

option go_package = "example.com/example/api/v2;api_v2";

// Define a service with versioned messages
service ExampleService {
  rpc GetExample(GetRequest) returns (GetResponse);
}

message GetRequest {
  // Optional field to be deprecated in the future.
  string optional_field = 1 [deprecated=true];
}

message GetResponse {
  string message = 1;
}
```

**Flow:**

1. API Change Request
2. Code Review
3. Semantic Versioning
4. Deprecation Planning
5. Runtime Validation
6. Continuous Integration
7. Canary Release
8. Production Deployment

