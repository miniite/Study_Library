---
type: "practice"
difficulty: "Deep"
---

# How do you secure a Schema Registry — auth for registration vs read, and preventing a rogue producer from overwriting a subject's compatibility mode?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Phased Schema Rollouts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Phased_Schema_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're tasked with securing a Schema Registry to prevent unauthorized access and ensure data integrity. A Schema Registry is crucial in maintaining schema consistency across producers and consumers in an event-driven architecture. We need to focus on two primary aspects: authentication for registration and read operations, as well as preventing rogue producers from overwriting the compatibility mode of subjects.

First, let's break down what a Schema Registry does:

It manages versioning and validation of schemas.
It ensures backward and forward compatibility when schema changes occur.
It acts as a central repository for defining schema versions.

Now, let’s dive into securing these aspects step by step:

#### Triaging the Question

Authentication Mechanisms:

We need to implement authentication mechanisms that differentiate between registration (writing) and read operations (retrieval).
The goal is to ensure that only authorized users can register new schemas or modify existing ones, while all consumers can safely read the latest version of a schema.

Compatibility Mode Management:

Preventing a rogue producer from altering the compatibility mode (e.g., backward, forward, or full) of an existing subject is critical to maintaining data integrity and backward compatibility.
This involves ensuring that changes to the compatibility mode are only allowed by authorized entities with specific permissions.

#### First Principles Solution

Authentication for Registration vs Read Operations:

Registration (Write Access): We need a robust authentication mechanism to verify that producers registering new schemas or modifying existing ones have the appropriate privileges.
Implement OAuth2, JWT tokens, or any other secure token-based system.
Ensure that these tokens carry necessary claims such as role, subject, and permissions to validate their legitimacy.
Read Operations (Retrieve Access): For read operations, a simpler authentication mechanism can be used since the primary concern is ensuring that consumers are only accessing the latest valid schemas.
Use API keys or basic HTTP authentication for minimal access controls.

Preventing Rogue Producers from Overwriting Compatibility Mode:

Implement role-based access control (RBAC) to restrict who can modify the compatibility mode of a subject.
Define roles such as SchemaAdmin, SchemaEditor, and SchemaViewer.
Ensure that only users with explicit SchemaAdmin or SchemaEditor roles can change the compatibility mode.
Use schema versioning policies:
Define rules in your Schema Registry to enforce strict validation of new schema versions based on the current compatibility mode.
Implement a review process where changes are approved by authorized users before they take effect.

#### Closing Summary

By implementing a robust authentication system and using RBAC, we can ensure that only authorized entities can register or modify schemas while also maintaining data integrity. The key is to carefully define roles and permissions, ensuring that read operations are accessible to all consumers but write operations (especially those involving compatibility mode changes) are restricted to trusted users.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to distinguish between authentication for registration vs read operations. Both require different levels of security.
Mistake 2: Overlooking the need for a review process before allowing compatibility mode changes, which can lead to data inconsistencies.

#### Follow-up
How would you handle legacy systems that do not support advanced authentication mechanisms? What about audit logging and monitoring?

**Comparison:**

```
Role	Permissions
SchemaAdmin	Register, modify schemas; change compatibility mode
SchemaEditor	Register, modify schemas; no change to compatibility mode
SchemaViewer	Retrieve latest schema version
```

**Code:**

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.post("/register_schema/")
async def register_schema(auth_token: str):
    # Validate the auth token against a trusted service
    if validate_token(auth_token) == False:
        raise HTTPException(status_code=401, detail="Unauthorized")

    # Register the schema based on validation logic
    await register_new_schema()
    return {"message": "Schema registered successfully"}

@app.get("/get_schema/")
async def get_schema(auth_token: str):
    if validate_token(auth_token) == False:
        raise HTTPException(status_code=401, detail="Unauthorized")

    # Retrieve the latest schema version based on validation logic
    schema_version = await retrieve_latest_schema()
    return {"schema": schema_version}
```

**Example input / output:**

```text
# Example of a valid registration request
curl -X POST \
  http://localhost:8000/register_schema/ \
  -H 'Authorization: Bearer <valid-auth-token>' \
  -d '{"schemaJson": "{\"type\":\"record\",\"name\":\"OrderEvent\",\"fields\":[{\"name\":\"id\",\"type\":\"long\"},{\"name\":\"amount\",\"type\":\"double\"}]}"}'

# Example of a valid read request
curl -X GET \
  http://localhost:8000/get_schema/ \
  -H 'Authorization: Bearer <valid-auth-token>'
```

**Data:**

```json
{
  "subjects": [
    {
      "name": "user-event",
      "compatibilityMode": "forward"
    },
    {
      "name": "order-event",
      "compatibilityMode": "full"
    }
  ],
  "schemas": [
    {
      "subject": "user-event",
      "version": 1,
      "schemaJson": "{\"type\":\"record\",\"name\":\"UserEvent\",\"fields\":[{\"name\":\"id\",\"type\":\"long\"},{\"name\":\"name\",\"type\":\"string\"}]}",
      "status": "registered"
    },
    {
      "subject": "order-event",
      "version": 1,
      "schemaJson": "{\"type\":\"record\",\"name\":\"OrderEvent\",\"fields\":[{\"name\":\"id\",\"type\":\"long\"},{\"name\":\"amount\",\"type\":\"double\"}]}",
      "status": "registered"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Producer sends SchemaRegistration Request
8. Auth Token Valid?
9. Register New Schema
10. Raise 401 Unauthorized
11. Schema Registry Logs Request
12. Respond with 401Unauthorized
13. Consumer Retrieves LatestSchema
14. Auth Token Valid?
15. Retrieve Latest Schema Version
16. Raise 401 Unauthorized
17. Schema Registry Logs Request
18. Respond with 401Unauthorized
19. Change Compatibility Mode
20. Auth Token Valid?
21. Validate Change; NotifySchema Admins if Needed
22. Raise 403 Forbidden
23. Update Subject's CompatibilityMode if Approved

