---
type: "practice"
difficulty: "Deep"
---

# Your image automation opened a PR but the image scan failed after merge — how do you fail the deploy safely and prevent the vulnerable image from reconciling?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Resilience to Git Outages](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When an automated pipeline opens a pull request (PR) to update container images, but the scan detects vulnerabilities post-merge, we need to ensure that the deployment fails safely and prevent any vulnerable image from reconciling into production. The key is to implement conditional logic in the GitOps process, such as using pre-deployment checks or hooks, to enforce a robust security policy.

#### Triaging the Question

The question revolves around ensuring the safety of deployments through GitOps practices by failing the deployment if a container image scan detects vulnerabilities. This involves understanding how GitOps workflows interact with CI/CD pipelines and how we can integrate security checks into our deployment process. We'll need to consider:

The current state of the GitOps workflow.
The integration points between CI/CD and GitOps.
Security policies for container images.

#### First Principles Solution
Pre-deployment Image Scans: Integrate a pre-deployment image scanning step in your CI/CD pipeline.
Conditional Deployment Logic: Use conditional logic to fail the deployment if vulnerabilities are detected.
GitOps Configuration Management: Ensure that GitOps is configured correctly to respect these policies.
Step-by-Step Solution

Pre-deployment Image Scans:

Integrate a tool like Clair, Aqua Security, or OpenSCAP into your CI/CD pipeline to scan images for vulnerabilities.
Run the image scans as part of the PR build process before merging.

Conditional Deployment Logic:

Use tools like Argo CD, Flux, or Spinnaker that support conditional deployment based on the state of the resources.
Implement a custom hook or policy in these tools to check for any security violations.
For example, using Argo CD, you can define a pre-apply hook that runs a script to verify image scan results.

GitOps Configuration Management:

Ensure your GitOps repositories (e.g., Helm charts, Kustomize files) are updated to reflect these security policies.
Use operators or custom resource definitions (CRDs) in Kubernetes to enforce these rules during reconciliation.

Fail-Safe Mechanisms:

If a vulnerability is detected post-merge, the GitOps tool should fail the deployment and not allow the vulnerable image to be reconciled into production.
Optionally, you can have an alerting system (e.g., Slack notifications) that sends immediate alerts to security teams upon detection of vulnerabilities.

Security Policies:

Define clear security policies for acceptable images based on vulnerability scores or other metrics.
Ensure these policies are documented and followed by all stakeholders involved in the deployment process.

#### Closing Summary

By implementing pre-deployment image scanning, conditional deployment logic, and fail-safe mechanisms, we can ensure that any potentially vulnerable container images do not make their way into production. This approach aligns with best practices in GitOps and DevSecOps to maintain a secure and reliable infrastructure.

#### Common Mistakes
and Follow-Ups
Mistake 1: Skipping pre-deployment scans due to time or resource constraints.

#### Follow-Up
Prioritize security by allocating necessary resources for image scanning.
Mistake 2: Failing to integrate conditional logic effectively, leading to manual overrides.

#### Follow-Up
Automate checks and enforce policies rigorously to avoid human error.
Mistake 3: Not having a robust alerting system in place.

#### Follow-Up
Implement real-time monitoring and alerts for quick response to security incidents.

**Comparison:**

```
Policy	Action	Condition
1	Allow	Vuln score < 50
2	Block	Vuln score >= 50
```

**Code:**

```bash
# Example pre-deployment image scanning script using Clair
#!/bin/bash

IMAGE_TO_SCAN=$1
VULNERABILITY_THRESHOLD=5  # Define a threshold score

# Run the scan
echo "Scanning $IMAGE_TO_SCAN for vulnerabilities..."
clair_scan_result=$(clair-scan "$IMAGE_TO_SCAN")

if [ $? -eq 0 ]; then
    echo "Scan successful."
else
    echo "Scan failed. Image may contain vulnerabilities."
fi

# Check if any vulnerability is above the threshold
vuln_details=$(echo $clir_scan_result | jq '.Vulnerabilities[] | select(.Score >= $VULNERABILITY_THRESHOLD)')
if [ -z "$vuln_details" ]; then
    echo "No high-risk vulnerabilities detected. Deployment allowed."
else
    echo "High-risk vulnerabilities found in image: $IMAGE_TO_SCAN"
    exit 1  # Fail the deployment
fi
```

**Example input / output:**

```text
Input (PR merged with a new image):
- New image tag: v1.3.0
- Image scan result: High risk vulnerability found

Output (GitOps reconcile outcome):
- Deployment failed due to high-risk vulnerabilities in the image.
- No vulnerable image reconciled into production.
```

**Data:**

```yaml
# Example Helm chart values.yaml with security policy enforcement
image:
  repository: example/image
  tag: latest

securityPolicies:
  - name: vulnerable-image-policy
    actions: ["allow", "block"]
    rules:
      - matchImage: true
        when:
          - vulns > 50 # Vuln score threshold
        then: block
```

**Flow:**

1. Yes
2. No
3. Trigger CI/CD pipeline
4. Run image scan
5. Vulnerability detected?
6. Fail deployment and notifysecurity team
7. Proceed with deployment

