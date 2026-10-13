---
type: "practice"
difficulty: "Deep"
---

# A blue-green deploy briefly runs two API versions with incompatible response shapes and clients see intermittent errors. How do you make rollout safe: contract compatibility, feature flags, and canary analysis?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ API Versioning and Deprecation](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Versioning_and_Deprecation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To ensure a smooth blue-green deployment where two API versions with incompatible response shapes coexist without causing client errors, we need to implement several strategies: contract compatibility, feature flags, and canary analysis. By carefully managing the rollout process, we can mitigate risks and provide a seamless transition.

#### Triaging the Question

The core of this question revolves around ensuring that clients consuming different API versions do not encounter unexpected behavior during a blue-green deployment. The key aspects to consider are:

Contract Compatibility: Ensuring that the APIs remain compatible throughout the deployment.
Feature Flags: Dynamically enabling new features or handling breaking changes.
Canary Analysis: Gradually introducing the new version and monitoring its impact before full rollout.

#### Assumptions
The current API is in use by clients, and a new version (v2) needs to be deployed alongside it (v1).
v2 has incompatible response shapes compared to v1.
We aim to maintain service availability during the transition period.

#### Solution
from First Principles
1. Contract Compatibility

To start, we need to ensure that both API versions are backward-compatible as much as possible. This involves:

API Versioning: Clearly define which parts of the API remain stable and which can change.
Breaking Changes Management: Document any breaking changes explicitly.

For instance, if v2 introduces new fields or changes data types, we need to provide a way for clients to adapt without immediate failure.

2. Feature Flags

Feature flags allow us to enable or disable specific features dynamically based on client requests. This approach is particularly useful when:

Rolling out New Features Gradually: By enabling the feature flag for certain clients, we can test it in production.
Handling Breaking Changes: Introduce new endpoints or methods gradually and monitor their impact.

Here’s how you might implement a feature flag mechanism:

PYTHON
class FeatureFlags:
    @staticmethod
    def is_v2_enabled(user_id):
        # Logic to determine if v2 should be enabled for the given user
        return random.random() < 0.1  # Example: 10% of users get v2

def handle_request(request, version):
    if version == 'v1':
        # Handle request using v1 API logic
        response = api_v1_handle_request(request)
    elif FeatureFlags.is_v2_enabled(user_id=request.user_id):
        # Handle request using v2 API logic with fallback to v1 if necessary
        try:
            response = api_v2_handle_request(request)
        except IncompatibleResponseError as e:
            response = handle_incompatibility_fallback(e, request)
    else:
        raise ValueError("Unsupported version")

    return response
3. Canary Analysis

Canary analysis involves deploying the new API version to a small subset of clients and monitoring its performance before full rollout. This helps in identifying potential issues early.

Steps for implementing canary analysis:

Select a Subset of Clients: Randomly select a small percentage (e.g., 5%) of active users.
Monitor Performance: Track metrics such as error rates, latency, and resource usage.
Rollback Mechanism: If issues arise, quickly revert to the previous version.

A sample implementation might look like this:

PYTHON
def canary_analysis(request, version):
    if random.random() < 0.05:  # 5% chance of being a canary user
        return handle_request(request, 'v2')
    else:
        return handle_request(request, 'v1')

#### Closing Summary
and Common Mistakes

To summarize, the key to making a blue-green deployment safe is through careful planning and implementation. By ensuring contract compatibility, using feature flags for controlled rollouts, and performing canary analysis, we can minimize disruptions and ensure a smooth transition.

#### Common Mistakes

Failing to document breaking changes properly.
Overlooking edge cases where clients might face unexpected behavior due to version incompatibility.
Not setting up proper monitoring and rollback mechanisms during the deployment process.

By addressing these issues comprehensively, you can mitigate risks and provide a robust deployment strategy for your API services.

**Comparison:**

```
Scenario	Version	Client Experience
Initial State	v1	Normal operations
Feature Flag	v2 enabled	Partial rollout
Canary Analysis	v2 used	Limited testing
```

**Code:**

```python
class FeatureFlags:
    @staticmethod
    def is_v2_enabled(user_id):
        return random.random() < 0.1

def handle_request(request, version):
    if version == 'v1':
        response = api_v1_handle_request(request)
    elif FeatureFlags.is_v2_enabled(user_id=request.user_id):
        try:
            response = api_v2_handle_request(request)
        except IncompatibleResponseError as e:
            response = handle_incompatibility_fallback(e, request)
    else:
        raise ValueError("Unsupported version")

    return response

def canary_analysis(request, version):
    if random.random() < 0.05:  # 5% chance of being a canary user
        return handle_request(request, 'v2')
    else:
        return handle_request(request, 'v1')
```

**Example input / output:**

```text
# Example input and output for canary_analysis function

request_v1 = {"method": "POST", "path": "/user/profile"}
response_v1 = handle_request(request_v1, 'v1') # Returns response from v1 API

request_canary = {"method": "POST", "path": "/user/v2/profile", "user_id": 123}
canary_response = canary_analysis(request_canary, 'v2') # 5% chance of handling as v2
```

**Data:**

```json
{
  "api_v1_handle_request": {
    "method": "POST",
    "path": "/user/profile"
  },
  "api_v2_handle_request": {
    "method": "POST",
    "path": "/user/v2/profile"
  },
  "handle_incompatibility_fallback": {
    "method": "POST",
    "path": "/fallback"
  }
}
```

**Flow:**

1. Yes
2. No
3. Client Request
4. Is Canary User?
5. Handle V2
6. Handle V1
7. Monitor Performance, Rollbackif Necessary
8. Error Handling

