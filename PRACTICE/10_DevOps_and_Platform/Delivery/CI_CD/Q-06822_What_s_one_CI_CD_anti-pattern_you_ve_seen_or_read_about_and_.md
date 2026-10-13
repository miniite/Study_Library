---
type: "practice"
difficulty: "Intermediate"
---

# What's one CI/CD anti-pattern you've seen (or read about) and how would you fix it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Schema Changes in Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Schema_Changes_in_Delivery_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the realm of Continuous Integration and Continuous Delivery, one common anti-pattern is "Feature Flag Hell," where multiple features are toggled on or off using various flags spread across the codebase without proper management. This leads to a spaghetti code structure, making it difficult to manage dependencies, rollbacks, and feature interactions.

#### Triaging the Question

The question probes your understanding of CI/CD practices, specifically focusing on how effective feature management can impact development processes. It also tests your ability to identify problematic patterns and propose actionable solutions.

#### First-Principles Solution

To address this anti-pattern, we need to establish a robust feature management strategy that integrates seamlessly with our CI/CD pipeline. Here’s a step-by-step approach:

Define Feature Toggles: Clearly define each toggle and ensure it is well-documented.
Centralize Management: Use a centralized feature toggling service like LaunchDarkly or Rollout.io to manage toggles across the codebase.
Automated Testing: Integrate unit, integration, and end-to-end tests that cover different feature states.
Rollback Mechanism: Implement automated rollback capabilities in case of issues post-deployment.
CI/CD Pipeline Integration: Ensure that the pipeline triggers based on specific toggle states to facilitate controlled rollouts.

#### Example
Workflow

Feature Toggles Definition:

Define a new feature as feature-x.
Document its purpose, dependencies, and expected behavior in a GitOps repository.

Centralize Management:

Deploy a feature toggling service.
Create a toggle for feature-x with appropriate configurations (e.g., default state, rollout percentage).

Automated Testing:

Write unit tests to validate the feature logic independently.
Develop integration and end-to-end tests that cover both enabled and disabled states.

Rollback Mechanism:

Implement rollback policies in your CI/CD pipeline.
Ensure that toggles can be flipped back or removed if issues arise during production.

CI/CD Pipeline Integration:

Configure the CI/CD pipeline to deploy only when specific toggles are enabled.
Use environment variables or secret management tools to pass toggle states securely.

#### Closing Summary
and Common Mistakes/Follow-Ups

Implementing a robust feature management strategy is crucial for maintaining code quality, ease of maintenance, and rapid development cycles. Common mistakes include not documenting features adequately, using hard-coded toggles, and failing to integrate rollback mechanisms. Always ensure that your CI/CD pipeline can handle feature toggles gracefully.

When discussing this in an interview, it's important to highlight the importance of centralization, automation, and thorough testing. Additionally, discuss how this approach aligns with broader DevOps principles and helps in managing complex deployments more effectively.

**Comparison:**

```
Feature	Description	Toggle State
Feature A	Enables new payment processing	on
Feature B	Adds support for multiple languages	off
Feature C	Enhances user interface elements	30%
```

**Code:**

```python
# Example Python code snippet for feature toggling using LaunchDarkly API
import requests

def get_feature_flag(feature_name):
    response = requests.get(f"https://api.launchdarkly.com/flags/{feature_name}")
    return response.json()['value']

if get_feature_flag("feature-x"):
    # Feature is enabled, proceed with logic
    pass
else:
    # Feature is disabled, skip or handle gracefully
    pass
```

**Example input / output:**

```
Input: Feature toggle state for feature-x is set to on.
Output: The system enables the new feature and proceeds with any necessary logic.
Example Dry Run
SHELL
Copy
# Assuming LaunchDarkly API returns "on"
curl -X GET https://api.launchdarkly.com/flags/feature-x

# Expected Response
{
  "value": true,
  "variation": 1,
  "segmentMatch": null,
  "clientIP": "192.0.2.1",
  "flagKey": "feature-x",
  "featureFlag": {
    "key": "feature-x",
    "name": "Feature X",
    "description": "Enables new feature X functionality.",
    "onPercentage": 10,
    "defaultState": "off",
    "variationIndex": 1
  }
}
```

**Data:**

```json
{
  "toggles": {
    "feature-x": {
      "name": "Feature X",
      "description": "Enables new feature X functionality.",
      "onPercentage": 10,
      "defaultState": "off"
    }
  }
}
```

**Flow:**

1. CI/CD Trigger
2. Feature Toggles API Call
3. Feature Logic Execution
4. Rollback Mechanism
5. Error Handling & Logging

