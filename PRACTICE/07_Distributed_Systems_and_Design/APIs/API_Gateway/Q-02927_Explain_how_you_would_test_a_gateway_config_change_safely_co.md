---
type: "practice"
difficulty: "Deep"
---

# Explain how you would test a gateway config change safely: contract tests, shadow traffic/mirroring, and diff testing between old and new routing before cutover.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Config Testing and Rollout](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Config_Testing_and_Rollout.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When updating an API Gateway configuration, ensuring a smooth transition is crucial to avoid disrupting production services. This process involves several steps: contract tests, shadow traffic/mirroring, and diff testing between old and new routing configurations before a full cutover. Let's break down each of these methods from first principles.

**Comparison:**

```
Test Type	Description
Contract Tests	Verify that the new configuration adheres to expected contracts (e.g., API responses, error handling) without breaking existing functionality.
Shadow Traffic/Mirroring	Route a portion of production traffic through the new config while monitoring for issues or discrepancies in performance and behavior.
Diff Testing	Compare routing configurations pre- and post-cutover to ensure that all paths and services are correctly mapped and function as expected.
```

**Code:**

```
Example code snippet for contract testing using Python

def test_contract(api_client): response = api_client.get('/api/v1/resources') assert response.status_code == 200 assert 'expected_field' in response.json()
```

**Example input / output:**

```text
Input:
- New Gateway Config: Updated routing rules, possibly including new service integrations.
Output:
- Successful Contract Tests on the new config without any breaking changes.
- Shadow Traffic Showing no discrepancies between old and new configurations.
- Diff Testing Results Highlighting differences in routing paths before full cutover.
```

**Flow:**

1. Initial Setup
2. Contract Tests
3. Mirroring Traffic
4. Diff Testing
5. Cutover
6. Monitor Production

