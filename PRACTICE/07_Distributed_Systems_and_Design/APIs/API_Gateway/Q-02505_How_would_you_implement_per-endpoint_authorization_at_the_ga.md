---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement per-endpoint authorization at the gateway from a policy file that maps method+path to required scopes, and how do you keep that mapping in sync with the actual API as it evolves?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Security and Data Masking](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Security_and_Data_Masking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement per-endpoint authorization at an API Gateway using a policy file, we need to ensure that each API endpoint is correctly mapped to its required scopes and that these mappings are dynamically updated when the API evolves. The core idea is to leverage a configuration-driven approach where the policy file acts as the authoritative source of truth for access control rules.

First, let's break down what the question is really asking:

Authorization Mappings: We need to map each endpoint (method + path) to the required scopes.
Policy File: This is our source configuration that defines these mappings.
Sync Mechanism: Ensure that the policy file stays in sync with the evolving API.

Now, let’s build this solution from first principles:

Step-by-Step Solution

Define the Policy File Structure:

The policy file should be structured as a JSON or YAML document where each endpoint is mapped to its required scopes.
YAML
api_endpoints:
  /users/{id}:
    GET: [user_read]
    POST: [user_write, user_admin]
  /admin/dashboard:
    GET: [admin_dashboard]

API Gateway Setup:

The API Gateway should read the policy file at startup and load the mappings into memory.
It should also have a mechanism to periodically refresh this configuration.

Authorization Middleware:

Implement middleware that intercepts requests, checks the method + path against the loaded policies, and verifies if the user has the required scopes.
PYTHON
import json

# Example policy file structure
with open('policy.json', 'r') as f:
    policy = json.load(f)

def authorize_request(path, method, scopes):
    for endpoint in policy['api_endpoints']:
        if path.startswith(endpoint) and method.upper() in policy['api_endpoints'][endpoint]:
            required_scopes = policy['api_endpoints'][endpoint][method.upper()]
            # Check user's token against the required scopes
            return all(scope in user_token_scopes for scope in required_scopes)
    return False

def gateway_handler(request, response):
    path = request.path
    method = request.method
    if not authorize_request(path, method, current_user_scopes):
        raise UnauthorizedError("Insufficient permissions")
    # Proceed with handling the actual API request

Dynamic Sync Mechanism:

Use a distributed configuration management system like Consul or Etcd to store and manage the policy file.
Implement a watcher that monitors changes in the policy file and reloads it when updates are detected.

#### Closing Summary

To summarize, implementing per-endpoint authorization at an API Gateway involves defining clear mappings between endpoints and required scopes, configuring the gateway to read these policies, and ensuring dynamic syncing with evolving APIs. This approach ensures secure access control while maintaining flexibility as the API evolves.

#### Common Mistakes/Follow-Ups
Not Using a Watcher: Failing to implement a mechanism that updates the policy file dynamically can lead to security vulnerabilities.
Lack of User Scopes Verification: Ensuring that user tokens contain all required scopes is crucial; otherwise, unauthorized access might occur.
Manual Configuration Management: Relying on manual file updates without automation increases the risk of misconfiguration.

**Comparison:**

```
Endpoint	GET	POST
/users/{id}	user_read	[user_write, user_admin]
/admin/dashboard	admin_dashboard	-
```

**Code:**

```python
import json

# Example policy file structure
with open('policy.json', 'r') as f:
    policy = json.load(f)

def authorize_request(path, method, scopes):
    for endpoint in policy['api_endpoints']:
        if path.startswith(endpoint) and method.upper() in policy['api_endpoints'][endpoint]:
            required_scopes = policy['api_endpoints'][endpoint][method.upper()]
            # Check user's token against the required scopes
            return all(scope in scopes for scope in required_scopes)
    return False

def gateway_handler(request, response):
    path = request.path
    method = request.method
    if not authorize_request(path, method, current_user_scopes):
        raise UnauthorizedError("Insufficient permissions")
    # Proceed with handling the actual API request
```

**Example input / output:**

```text
# Sample Input
request.path = '/users/123'
request.method = 'GET'

# Sample Output
# If the current_user_scopes contains ['user_read'], then authorization is successful.
# Otherwise, a 401 Unauthorized error will be raised.
```

**Data:**

```yaml
api_endpoints:
  /users/{id}:
    GET: [user_read]
    POST: [user_write, user_admin]
  /admin/dashboard:
    GET: [admin_dashboard]
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Request Received
6. Is Path and Method Mapped?
7. Check User Scopes AgainstPolicy
8. User Has Required Scopes?
9. Proceed with API RequestHandling
10. Reject Request: UnauthorizedError
11. Return 405 Method NotAllowed

