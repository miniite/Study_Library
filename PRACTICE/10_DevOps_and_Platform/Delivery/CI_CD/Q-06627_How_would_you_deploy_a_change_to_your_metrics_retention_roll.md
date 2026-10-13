---
type: "practice"
difficulty: "Intermediate"
---

# How would you deploy a change to your metrics retention/rollup so that detectors relying on long baselines aren't starved of history after the change?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rollback Strategy](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

The key challenge here is ensuring that new versions of our metrics processing pipelines don’t disrupt existing detector configurations, which rely on historical data spanning long baselines. We need to deploy changes in a way that minimizes downtime and ensures smooth operation.

#### Triage

The question revolves around maintaining the integrity of continuous integration and deployment (CI/CD) processes while making updates to how we retain and roll up metrics. Specifically, we must ensure that detectors, which rely on long baselines for accurate functioning, aren’t deprived of historical data after a code change or configuration update.

#### Assumptions
We are dealing with a large-scale monitoring system where detectors operate on long historical periods.
Metrics retention policies can be updated via CI/CD processes.
The detectors must receive enough historical data to function correctly without false positives or negatives.

#### First Principles Solution

Versioning and Canary Deployments:

Implement versioned deployments to allow rolling out new versions of the metrics processing pipelines gradually rather than all at once.
Use canary releases where a small percentage of the detectors start using the updated pipeline, and monitor their behavior before fully deploying.

Historical Data Backfilling:

Ensure that historical data is retained or backfilled as part of any deployment process to support long baselines for detectors.
Use batch processing jobs to populate new metrics storage with relevant historical data from old stores.

Synchronization Mechanisms:

Implement synchronization between the new and existing pipelines to ensure both versions coexist temporarily until a smooth transition is complete.
This might involve writing intermediate states or using a hybrid approach where new and old processes communicate during the overlap period.

Monitoring and Alerts:

Set up robust monitoring to detect any issues early, such as missing historical data leading to false detections.
Define alert thresholds for critical metrics that can trigger immediate rollback if needed.

Automated Testing:

Develop automated tests to verify that both old and new pipelines produce consistent results during the transition period.
Use test cases that simulate different failure modes to ensure resilience against unexpected issues.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

By employing canary deployments, historical data backfilling, synchronization mechanisms, monitoring, alerts, and automated testing, we can mitigate risks associated with deploying changes in metrics retention policies. Common mistakes include not having a rollback plan or insufficient monitoring, which can lead to prolonged disruptions or false alarms.

Additional Considerations:
Performance Impact: Ensure that the new pipeline does not introduce performance bottlenecks.
User Experience: Minimize any impact on end-users who rely on these detectors for critical operations.
Documentation: Maintain clear documentation of the deployment process, including steps and expected outcomes.

**Comparison:**

```
Deployment Stage	Action
Canary Release	Deploy 5% of detectors to test new pipeline behavior
Backfill Data	Populate new storage with historical data from legacy system
Monitoring	Monitor detector outputs for false positives/negatives
Full Rollout	Deploy to remaining 95% if no issues detected
```

**Code:**

```python
# Example Canary Deployment Script
def canary_deployment(deploy_version):
    if deploy_version % 10 == 0:
        # Deploy to a small percentage of detectors (e.g., 5%)
        pass
    else:
        # Deploy to the rest of the detectors
        pass
```

**Example input / output:**

```text
Input:
- Deploy new metrics retention policy with a 30-day retention period and legacy backfill

Output:
1. Canary release of the new pipeline to 5% of detectors
2. Backfill historical data for new storage from legacy system
3. Monitor performance and detector outputs
4. Full deployment if no issues detected, otherwise rollback
```

**Data:**

```json
{
  "metrics_storage": {
    "retention_policy": "30_days",
    "backfill_data_source": "legacy_system"
  },
  "detector_configurations": [
    {
      "name": "CPU_Underutilization",
      "baseline_period": "90_days",
      "thresholds": [80, 60]
    }
  ]
}
```

**Flow:**

1. Canary Release
2. Backfill Historical Data
3. Monitor Detectors
4. Roll Out if Stable
5. Full Deployment Complete
6. Rollback if Issues Detected

