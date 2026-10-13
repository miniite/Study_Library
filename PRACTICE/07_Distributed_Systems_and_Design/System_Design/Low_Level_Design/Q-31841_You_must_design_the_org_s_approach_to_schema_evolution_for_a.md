---
type: "practice"
difficulty: "Expert"
---

# You must design the org's approach to schema evolution for an internal enum exposed over APIs; a new value broke old clients doing exhaustive switches; how do you make "adding a value" non-breaking by design and mandate a graceful-degradation contract on consumers?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Domain Event Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/Domain_Event_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of schema evolution, particularly when dealing with enums exposed over APIs, we need to ensure that adding new values is non-breaking while gracefully degrading existing clients. This involves a strategic low-level design (LLD) that mandates versioned schemas and dynamic type handling.

First, let's triage the question:

Core Question: How do you handle adding an enum value in a way that doesn't break old clients?
Key Probing Points:
How to maintain backward compatibility.
Ensuring graceful degradation without disrupting existing functionality.
Managing versioning of schemas.

#### Assumptions

The system is exposed via REST APIs, and enums are used extensively.
Existing clients rely on exhaustive switches over the enum values.
A new value has been added that broke old client implementations.

Now, let's reason from first principles to build a robust solution:

Step-by-Step Solution

Versioning Schemas:

Introduce schema versioning where each version of your API contract is tied to a specific schema definition.
Clients should be able to negotiate the version they support at runtime, allowing them to fall back to earlier versions if needed.

Dynamic Enum Handling:

For enums, use dynamic types that can handle new additions without breaking existing logic.
Implement a fallback mechanism for enums where unhandled values are gracefully processed or ignored.

Client-side Fallback Mechanisms:

Ensure clients can handle changes by versioning their enum handling logic.
Provide clear documentation and examples to help developers update their implementations.

Server-side Validation and Logging:

On the server side, log any unexpected enum values that don't match known versions or schemas.
Validate incoming requests against current schema definitions to ensure only supported values are processed.

Testing and Simulations:

Regularly run integration tests with simulated clients using different versions of the API.
Use canary releases to introduce new schemas and monitor how they behave in production environments.

#### Implementation
Example
Server
Yes
No
Client Requests New Value
Is Versioned SchemaSupported?
Process Request Normally
Log Warning, Fallback toDefault Behavior
Return Graceful Error Response
Handle Unhandled Enums
Validate Incoming RequestsAgainst Current Schema
Notify Devs of BreakingChanges

#### Closing Summary

By implementing a versioned schema approach and dynamic enum handling, we ensure that adding new enum values does not break existing clients. This strategy mandates graceful degradation through clear fallback mechanisms and robust validation on the server side. Regular testing and logging help in maintaining system stability and identifying issues early.

#### Common Mistakes
and Follow-Ups

Mistake: Ignoring schema versioning, leading to breaking changes.

#### Follow-Up
Always version your API contracts.

Mistake: Not providing fallback mechanisms for clients.

#### Follow-Up
Ensure clients have clear guidance on handling unknown enum values.

Mistake: Failing to validate requests on the server side.

#### Follow-Up
Implement comprehensive validation and logging to catch unexpected values early.

**Comparison:**

| Schema Version | Supported Enum Values |
|----------------|----------------------|
| v1             | A, B                 |
| v2             | A, B, C              |

**Code:**

```python
# Example of a dynamic enum class in Python

class DynamicEnum:
    def __init__(self, value):
        self.value = value

    @staticmethod
    def validate(value):
        valid_values = ['A', 'B', 'C']
        if value not in valid_values:
            return False, f"Invalid value: {value}"
        return True, None

class MyEnum(DynamicEnum):
    A = DynamicEnum('A')
    B = DynamicEnum('B')
    C = DynamicEnum('C')

# Example usage
def process_request(enum_value):
    is_valid, error_msg = MyEnum.validate(enum_value)
    if not is_valid:
        print(error_msg)
        return 'GRACEFUL_ERROR_RESPONSE'
    # Process request normally
    return f"Processed {enum_value}"
```

**Example input / output:**

```text
Input:
- Client sends a request with enum value 'D'

Output:
- Server logs warning: Invalid value: D
- Returns graceful error response: GRACEFUL_ERROR_RESPONSE

Logging Example (Server-side):
2023-10-04 10:00:00 - WARNING - Invalid value: D received in request
```

**Data:**

```json
{
  "valid_values": ["A", "B", "C"]
}
```

**Flow:**

1. Server
2. Yes
3. No
4. Client
5. Process Request Normally
6. Fallback to Default Behavior
7. Return Graceful Error Response
8. Handle Unhandled Enums
9. Validate Incoming RequestsAgainst Current Schema

