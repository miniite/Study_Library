---
type: "practice"
difficulty: "Deep"
---

# A Kustomize overlay in production silently stopped applying a patch after a base upgrade — how would you catch this class of drift before it hits prod?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Reconciliation Failures](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In GitOps, maintaining consistency between development and production environments is crucial. When using Kustomize overlays in production, silent failures can occur if changes are not properly reflected or applied due to misconfigurations or drifts. To catch such issues before they hit the production environment, we need a comprehensive monitoring and validation strategy. This involves continuous integration, automated testing, and real-time monitoring of the state of our clusters.

#### Triage
The question is probing how to detect and mitigate silent failures in GitOps setups using Kustomize overlays. Key assumptions include that the base image or configuration has been updated, yet the overlay did not apply the intended changes correctly, leading to a silent failure. The goal is to identify such issues early to prevent them from affecting production.

#### First-principles Solution

Version Control and Patch Management: Ensure all Kustomize overlays are version-controlled in Git. This helps track changes and their application history.
Automated Testing: Implement unit tests, integration tests, and end-to-end tests for the Kustomize overlays to verify that they correctly apply the intended patches.
Pre-merge Validation: Use tools like kpt or a custom script to validate the overlay configuration before merging it into the main branch. This ensures that the overlay applies as expected on the latest base image.
Real-Time Monitoring: Deploy monitoring agents in your clusters to track the state of deployed overlays and compare them with desired configurations. Tools like Prometheus, Grafana, or Kubernetes-native metrics can provide insights into drifts.
Post-merge Validation: Implement CI/CD pipelines that run kustomize against the latest base image post-merge to ensure the overlay applies correctly.

#### Closing Summary
To catch silent failures in Kustomize overlays before they affect production, we need a layered approach involving version control, automated testing, pre-merge and post-merge validation, and real-time monitoring. This ensures that changes are thoroughly tested and validated before deployment, reducing the risk of silent drifts.

#### Common Mistakes/Follow-Ups

Overlooking Pre-Merge Validation: Not validating overlays against the latest base image can lead to unexpected behavior in production.
Insufficient Monitoring: Relying solely on post-deployment monitoring without real-time checks can miss critical issues during critical periods like rolling updates or emergency changes.
Neglecting Testing: Automated testing should cover various scenarios, including edge cases and complex configurations.

**Comparison:**

```
Test Type	Description
Unit Tests	Verifies individual components of the overlay.
Integration Tests	Ensures that different parts work together as expected.
End-to-End Tests	Simulates real-world scenarios and verifies the entire process from development to production.
```

**Code:**

```bash
# Example of a pre-merge validation script using kpt
#!/bin/bash

BASE_IMAGE="quay.io/myorg/base-image:latest"
OVERLAY_PATH="./overlays/staging"

if ! kustomize build $OVERLAY_PATH | kpt cfg set-values -; then
  echo "Validation failed. Overlay did not apply as expected."
  exit 1
fi
echo "Overlay validated successfully against base image."
```

**Example input / output:**

```text
$ kustomize build ./overlays/staging | kpt cfg set-values -
[Validation Output]
```

**Data:**

```yaml
# Example of a Kustomization.yaml file for an overlay
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ./base.yaml

patchesStrategicMerge:
- patch1.yaml
- patch2.yaml
```

**Flow:**

1. Check Base Image
2. Validate Overlay
3. Run CI/CD Pipeline
4. Real-Time Monitoring inProduction
5. Detect Drifts and Failures

