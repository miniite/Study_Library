---
type: "practice"
difficulty: "Deep"
---

# You're deciding how the org models 'a computed/denormalized read model for the frontend' (a dashboard aggregate) so the backend owns the shape; how do you version the payload so an old app client tolerates a new field being added or an optional field going missing?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Maintaining Denormalized Views](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/Maintaining_Denormalized_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a low-level denormalized read model that supports backward compatibility in a dashboard aggregate, we need to focus on how the backend can safely add or remove fields without breaking existing client applications. This involves choosing an appropriate versioning strategy and ensuring the payload structure is flexible enough to handle changes over time.

#### Triaging the Question

The core of this question revolves around maintaining backward compatibility when updating the schema of a read model that’s used by frontend dashboards. We need to consider how to version the payload so that old client applications can still function correctly even if new fields are added or existing ones removed from the backend.

#### Assumptions
Client Applications: These could be different versions of an application, possibly running in production and requiring support for multiple schema versions.
Backend Flexibility: The backend must be able to handle requests with potentially outdated schema versions without breaking down.
Read-Optimized Models: These models are designed primarily for read operations rather than write operations.

#### First-Principles Solution

To achieve backward compatibility, we can employ several strategies:

Schema Versioning: Ensure the payload includes a version identifier and use this to determine how to interpret the data.
Optional Fields Handling: Design fields as optional so that they may be omitted without breaking the overall structure.
Dynamic Data Parsing: Use dynamic parsing techniques to handle different schema versions gracefully.
Step-by-Step Solution

Version Identification:

Include a version field in the payload to identify which version of the schema is being used.

Optional Fields:

Design fields as optional by setting them with default values or using null values where appropriate. This allows for graceful handling when fields are absent.

Dynamic Parsing Logic:

Implement logic on both the backend and frontend to dynamically parse the payload based on its version.

Fallback Mechanisms:

Use fallback mechanisms to handle unknown schema versions, ensuring that any unexpected changes do not crash the system.

#### Example
Payload
JSON
{
  "version": "1.2",
  "dashboard_data": {
    "metric1": "value1",
    "metric2": "value2",
    "optional_metric3": null,
    "timestamp": "2023-10-09T14:48:00Z"
  }
}

#### Implementation
Example

Here's a simple implementation in JavaScript for parsing the payload:

JAVASCRIPT
function parsePayload(payload) {

#### cons
t version = payload.version;

#### cons
t { dashboard_data } = payload;

  switch (version) {
    case '1.2':
      return {
        metric1: dashboard_data.metric1,
        metric2: dashboard_data.metric2,
        optional_metric3: dashboard_data.optional_metric3 || null, // Fallback to null if missing
      };
    default:
      throw new Error(`Unknown version: ${version}`);
  }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, the key is in structuring the payload with a version identifier and optional fields, and implementing dynamic parsing logic to handle different versions gracefully. This approach ensures that old client applications can continue functioning even as new features are added or existing ones removed.

#### Common Mistakes

Forgetting to include a version field.
Not handling fallbacks for unknown schema versions.
Ignoring the impact of adding optional fields on older clients.

#### Follow-Ups

How would you handle scenarios where multiple fields need to be updated simultaneously?
What are some potential security implications of this approach, and how can they be mitigated?

**Comparison:**

```
Field	Version 1.1	Version 1.2	Notes
metric1	Present	Present	Always present
metric2	Present	Present	Always present
optional_metric3	Missing	Optional, null if missing	Can be added or removed
```

**Code:**

```javascript
function parsePayload(payload) {
  const version = payload.version;
  const { dashboard_data } = payload;

  switch (version) {
    case '1.2':
      return {
        metric1: dashboard_data.metric1,
        metric2: dashboard_data.metric2,
        optional_metric3: dashboard_data.optional_metric3 || null, // Fallback to null if missing
      };
    default:
      throw new Error(`Unknown version: ${version}`);
  }
}
```

**Example input / output:**

```text
Input:
{
  "version": "1.2",
  "dashboard_data": {
    "metric1": "value1",
    "metric2": "value2",
    "optional_metric3": null,
    "timestamp": "2023-10-09T14:48:00Z"
  }
}

Output:
{
  metric1: 'value1',
  metric2: 'value2',
  optional_metric3: null
}
```

**Data:**

```json
{
  "version": "1.2",
  "dashboard_data": {
    "metric1": "value1",
    "metric2": "value2",
    "optional_metric3": null,
    "timestamp": "2023-10-09T14:48:00Z"
  }
}
```

**Flow:**

1. Yes
2. No
3. Receive Payload
4. Is Version Known?
5. Parse Based on Version
6. Throw Error: Unknown Version
7. Return Data Object

