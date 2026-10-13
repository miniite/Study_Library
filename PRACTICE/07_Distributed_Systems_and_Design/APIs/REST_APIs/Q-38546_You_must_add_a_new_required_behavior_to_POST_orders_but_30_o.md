---
type: "practice"
difficulty: "Expert"
---

# You must add a new required behavior to POST /orders but 30% of traffic still comes from an app version you cannot force-upgrade. How do you evolve the contract without a hard break, and how long do you keep the compatibility shim?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ API Versioning and Deprecation](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Versioning_and_Deprecation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When adding a new required behavior to the POST /orders API while maintaining compatibility with an older app version that can't be forced to upgrade, we need to carefully design our solution to ensure backward compatibility without breaking existing functionality. The key is to evolve the API contract in a way that allows both old and new clients to coexist seamlessly. Let's break down the problem step by step.

#### Triage

The core question here involves designing an API endpoint (POST /orders) where we need to add a required field (let's call it newField). However, 30% of traffic comes from an older version of the app that cannot be upgraded. The goal is to update the backend to enforce this new requirement while keeping the old clients functional until they are eventually upgraded.

#### Assumptions
Old Clients: These are legacy apps that may not support newField and will send requests without it.
New Clients: These apps can be upgraded to include the new field but we cannot force them all at once.
Traffic Distribution: 30% of traffic is from old clients, while 70% comes from new or upgradeable clients.

#### Solution

To achieve this, we need a strategy that introduces backward compatibility for the old clients and ensures the new clients are required to provide newField. Here’s how we can do it:

Introduce an Optional Field with a Default Value: For now, add newField as optional with a default value in the API response schema.

Update the Backend Logic:

If newField is present in the request from new clients: proceed normally and validate its presence.
If newField is absent or not provided by old clients, handle it gracefully to maintain compatibility.

Graceful Degradation: Implement logic on the backend that checks if newField is missing for requests coming from known older client versions (e.g., via a user-agent header). For such requests, you can either log an error and return an appropriate HTTP status code or handle it in a way that doesn’t break the transaction.

#### Implementation
Steps

Schema Evolution:

JSON
{
  "order": {
    "id": "string",
    "items": [
      {
        "sku": "string",
        "quantity": number,
        "newField": { // Optional field with a default value
          "value": boolean, // Default: false
          "defaultMessage": "Field is optional and has a default value."
        }
      }
    ]
  }
}

Backend Logic:

PYTHON
from flask import request

def process_order(order):
    new_field_value = order.get('newField', {}).get('value')

    if not new_field_value and is_old_client(request.headers):  # Assume this function checks the user-agent for old clients
        return "400 Bad Request: newField is required.", 400

    # Proceed with normal processing
    ...

Monitoring and Gradual Transition:

Monitor the error logs to track when and if old clients start sending requests containing newField.
Once a sufficient number of old clients have been upgraded, remove the compatibility shim.

#### Closing Summary

By introducing a default value for newField in the response schema and handling missing fields gracefully on the backend, we can ensure backward compatibility. This allows us to gradually phase out support for the older client versions without immediately breaking their functionality. The transition period depends on how quickly old clients can be upgraded but should be managed carefully to minimize disruption.

#### Common Mistakes/Follow-Ups
Forgetting to Handle Edge Cases: Ensure that both old and new clients are properly accounted for.
Over-Engineering the Solution: While it’s important to handle compatibility, overcomplicating the solution can introduce unnecessary complexity. Keep it simple but effective.
Not Monitoring: Regularly monitoring error logs and usage patterns helps in making informed decisions about when to remove compatibility shims.

**Comparison:**

```
Client Version	Is Old Client?
v1.0	Yes
v2.0+	No
```

**Code:**

```python
from flask import request

def process_order(order):
    new_field_value = order.get('newField', {}).get('value')

    if not new_field_value:
        client_version = get_client_version(request.headers)
        if is_old_client(client_version):  # Assume this function checks the user-agent for old clients
            return "400 Bad Request: newField is required.", 400

    # Proceed with normal processing
    ...
```

**Example input / output:**

```text
# Example Input & Output
request = {
    'order': {
        'id': '12345',
        'items': [
            {'sku': 'A123', 'quantity': 1, 'newField': {'value': False}}
        ]
    },
    'headers': {'User-Agent': 'OldClient/1.0'}  # Simulate an old client
}

response = process_order(request['order'])

print(response)  # Output: "400 Bad Request: newField is required."
```

**Data:**

```json
{
  "order": {
    "id": "12345",
    "items": [
      {
        "sku": "A123",
        "quantity": 1,
        "newField": { // Optional field with a default value
          "value": false, // Default: false
          "defaultMessage": "Field is optional and has a default value."
        }
      }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Request arrives
4. Is client old?
5. Return error 400
6. Process normally

