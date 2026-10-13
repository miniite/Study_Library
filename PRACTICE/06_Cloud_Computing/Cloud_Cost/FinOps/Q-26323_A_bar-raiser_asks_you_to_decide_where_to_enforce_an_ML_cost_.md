---
type: "practice"
difficulty: "Expert"
---

# A bar-raiser asks you to decide where to enforce an ML cost policy: at the IaC/job-submission stage, at the Kubernetes admission-controller stage, or via cloud org policy. Walk me through the failure modes of each layer for GPU-cost governance and when you'd defense-in-depth across all three.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost-aware Defaults](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost-aware_Defaults.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address where to enforce an ML cost policy—specifically, GPU-cost governance—at different stages in a cloud environment (IaC/job-submission stage, Kubernetes admission-controller, and cloud org policy), we need to consider the failure modes of each layer comprehensively. The goal is to ensure robust, multi-layered security and compliance while understanding trade-offs related to complexity, enforceability, and operational impact.

IaC/Job-Submission Stage:

Key Idea: Enforcing policies at this stage can catch issues before resources are provisioned or jobs start running.
Failure Modes:
Insufficient Enforcement: Policies may be too permissive, allowing excessive GPU usage without detection until the job runs.
Complexity and Overhead: Implementing strict IaC policies can add significant complexity to development and deployment processes.
Limited Real-Time Control: Issues might only become apparent after resource allocation has occurred.

Kubernetes Admission-Controller Stage:

Key Idea: Admission controllers can enforce rules before resources are created, but they rely on the Kubernetes API being accessed correctly.
Failure Modes:
API Misuse: Developers or attackers might bypass admission controllers by directly interacting with the Kubernetes API.
Maintenance Overhead: Maintaining and updating admission controllers require significant effort and expertise.
False Positives/Negatives: Policy rules can be misconfigured, leading to either blocking legitimate requests (false positives) or failing to block malicious ones (false negatives).

Cloud Org Policy:

Key Idea: Cloud org policies provide a high-level way to enforce governance and compliance at the organizational level.
Failure Modes:
Granularity Issues: Policies might be too coarse-grained, lacking the necessary specificity for fine-tuning GPU usage.
Inflexibility: Changes to organization-wide policies can impact multiple teams, requiring coordination and potentially causing delays in development.
Delayed Feedback: Enforcement of org-level policies often results in delayed feedback loops, making it difficult to correct misuse immediately.

Defense-in-Depth Across All Three Layers:

By enforcing GPU-cost governance at all three layers, we achieve a more robust system that addresses different failure modes.
IaC/job-submission policies ensure no misconfigured or overly permissive jobs are run.
Kubernetes admission controllers provide an additional layer of security by preventing misuse before resources are allocated.
Cloud org policies enforce broader governance rules and compliance at the organizational level.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The most critical failure mode to avoid is missing any enforcement point, as it can lead to uncontrolled GPU usage, costing your organization significant money.
A common mistake is focusing too much on one layer while neglecting others. For instance, overly relying on IaC without complementing with admission controllers and org policies could leave gaps in governance.

#### Follow-up
questions might include specific use cases or scenarios where certain layers would be more appropriate, such as real-time monitoring requirements for admission controllers.

**Comparison:**

```
Layer	Enforce Point	Probing	Common Mistakes
IaC/Job Submission	Before resource allocation	Detect misconfigurations	Too permissive policies
Admission Controller	Before resource creation	Prevent misuse	API bypass, false positives/negatives
Cloud Org Policy	At the organization level	Enforce governance	Granularity issues, organizational delays
```

**Code:**

```
Example of an IaC policy snippet (Terraform)

resource "aws_iam_policy" "ml_cost_policy" { name = "ml-cost-policy" description = "Policy to govern GPU costs in ML jobs"

policy = jsonencode({ Version = "2012-10-17", Statement = [ { Action = ["ec2:RunInstances", "eks:CreateCluster"], Effect = "Deny", Resource = "*", Condition = { Bool = { "aws:RequestTag/MLJobType" : "NotEqualsIgnoreCase" "Research" } } }, ] }) }

CODE
Copy

```yaml
# Example Kubernetes Admission Controller config
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: gpu-cost-pod-security
spec:
  allowedUnsafeSysctls: [""]
  allowHostDirVolumeMounts: false
  requiredDropCapabilities:
  - ALL
  volumes:
    - configMap
    - downwardAPI
    - emptyDir
    - secret
  fsGroup:
    rule: RunAsAny
  fsGroupRangeRules:
  - ranges:
      - min: 1000
        max: 9999
    kind: RangeMinimum
  hostIPC: false
  hostNetwork: false
  hostPID: false
  imagePullPolicy: Always
  allowedCSRs:
  privileged: false
  runAsUser:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  user:
    rule: RunAsAny
YAML
Copy
# Example Cloud Org Policy (GCP)
resource "google_project_policy" "gpu_cost_limitation" {
  project = var.project_id

  bindings {
    role = "roles/resourcemanager orgPolicy"
    members = ["allUsers"]
    constraint = "compute.instance-allow-list"
    boolean_value = false
  }
}
```

**Example input / output:**

```text
Input:
- GPU usage policy enforcement points: IaC, admission controller, and cloud org policies.

Output:
- IaC/job-submission stage ensures misconfigurations are caught early.
- Kubernetes admission controllers provide real-time control but require careful management.
- Cloud org policies enforce broader governance at an organizational level, providing a comprehensive view.
```

**Data:**

```json
{
  "cloud_cost_policy": {
    "iac_job_submission": [
      {
        "name": "deny-research-jobs",
        "condition": {
          "tag": { "value": "Research", "op": "not-equals" },
          "action": [ "ec2:RunInstances", "eks:CreateCluster" ]
        }
      }
    ],
    "kubernetes_admission_controller": [
      {
        "name": "gpu-cost-control",
        "rules": [
          { "allowedSysctls": [], "deniedSystemCgroups": [] },
          { "allowedHostDirVolumeMounts": false, "allowedHostPaths": [] },
          { "allowedPrivilegedContainers": false, "allowedHostNetwork": false }
        ]
      }
    ],
    "cloud_org_policy": [
      {
        "name": "gpu_limitation",
        "constraint": "compute.instance-allow-list",
        "value": false
      }
    ]
  }
}
```

**Flow:**

1. Job Submission
2. IaC Check
3. Kubernetes AdmissionController
4. Cloud Org Policy Enforcement

