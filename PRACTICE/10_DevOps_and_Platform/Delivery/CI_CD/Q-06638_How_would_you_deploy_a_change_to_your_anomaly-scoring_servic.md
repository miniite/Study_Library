---
type: "practice"
difficulty: "Deep"
---

# How would you deploy a change to your anomaly-scoring service behind a feature flag so you can score in shadow, compare to the live path, and flip only when confident?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Feature Flags and Deploy Risk](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Feature_Flags_and_Deploy_Risk.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To deploy a change to an anomaly-scoring service behind a feature flag while scoring in shadow, comparing to the live path, and flipping the switch only when confident, we need to ensure that our deployment strategy is robust and minimizes risk. This involves understanding the current CI/CD pipeline, implementing feature flags, setting up A/B testing, and establishing a confidence threshold for production release.

**Comparison:**

| Feature Flag | Key          | Enabled |
|--------------|--------------|---------|
| Anomaly Scoring | anomaly_scoring | false   |
| ...           | ...          | ...     |

**Code:**

```python
# Example of a simple feature flag implementation in Python
class FeatureFlag:
    def __init__(self, key):
        self.key = key

    def is_enabled(self):
        # This should be replaced with actual flag logic, e.g., reading from a config or database
        return True  # Mocking for simplicity

# Example usage
feature_flag = FeatureFlag("anomaly_scoring")
if feature_flag.is_enabled():
    print("Feature enabled: Performing anomaly scoring in shadow.")
else:
    print("Feature disabled: Skipping anomaly scoring.")
```

**Example input / output:**

```

Given the initial setup where anomaly_scoring is disabled, here’s a dry-run:

PLAINTEXT
Copy
Input: Check if anomaly scoring should be performed in shadow.
FeatureFlag configuration: {'key': 'anomaly_scoring', 'enabled': False}
Output: Feature disabled: Skipping anomaly scoring.

Step 1: Enable the feature flag temporarily for testing.
FeatureFlag configuration updated to {'key': 'anomaly_scoring', 'enabled': True}

Input: Perform anomaly scoring in shadow mode.
Output: Performing anomaly scoring in shadow.

Step 2: Compare results with live path.
Anomaly Scoring (Shadow): [0.85]
Live Anomaly Scoring: [0.90]

Step 3: If the difference is within acceptable bounds, proceed to production.
If not, revert changes and further investigate issues.
```

**Data:**

```yaml
# Example configuration snippet for feature flags
feature_flags:
  - key: "anomaly_scoring"
    enabled: false
```

**Flow:**

1. No
2. Yes
3. Similarity Within Bounds
4. Difference ExceedsThreshold
5. Initial State - Feature Disabled
6. Is Shadow Mode Enabled?
7. Skip Anomaly Scoring
8. Perform Anomaly Scoring inShadow Mode
9. Compare Results with Live Path
10. Enable Feature Flag inProduction
11. Test Further, No Change forNow

