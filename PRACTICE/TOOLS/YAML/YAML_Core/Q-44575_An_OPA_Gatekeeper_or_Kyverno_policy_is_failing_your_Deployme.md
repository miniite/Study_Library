---
type: "practice"
difficulty: "Deep"
---

# An OPA/Gatekeeper or Kyverno policy is failing your Deployment YAML for a label you swear is present. How do you debug whether the policy reads the field path differently than you expect?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kustomize Patching](<../../../../TOOLS/YAML/YAML_Core/Kustomize_Patching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When debugging why an Open Policy Agent (OPA) or Gatekeeper/Kyverno policy might be failing a Deployment YAML, it's essential to ensure that the label in your Deployment matches how the policy is reading and validating it. This involves a series of steps to systematically verify the configuration.

#### Triage

The question revolves around understanding why an OPA/Gatekeeper policy is not recognizing a label as expected within your Deployment YAML. The key aspects are:

Policy Logic: How the policy reads and validates the field.
Field Path: The exact path used in the policy to access the label.
Input Data: The actual content of the Deployment YAML.
First Principles
Review Policy Configuration: Understand the specific rules defined in the OPA/Gatekeeper policy that are being applied to the Deployment.
Check Field Paths: Ensure that the field path used in the policy exactly matches how the label is structured and named in your YAML.
Debug with Example Data: Use an example input to run against the policy to see where it fails.
Step-by-Step Solution

Examine the Policy Configuration:

Open the OPA/Gatekeeper policy file and identify the rule or constraint that is failing.
Look for any match or allow statements, specifically focusing on the field path used.

Identify the Field Path:

Determine the precise field path used in the policy to access the label. For example, if your Deployment YAML has a label app: MyApp, the policy might use spec.metadata.labels.app.

Check Deployment YAML Structure:

Verify that the label is correctly defined in your Deployment YAML.
Ensure there are no typos or unexpected characters.

Run an Example Against the Policy:

Create a minimal example of the problematic Deployment YAML and run it against the policy using opa eval (for OPA) or similar tools provided by Gatekeeper/Kyverno.

#### Example
Code
YAML
# Minimal Example Deployment YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  template:
    metadata:
      labels:
        app: MyApp
Execute the Policy Check:
For OPA, use opa eval with the policy file and minimal example YAML.
BASH
opa eval --data deployment.yaml --policy my-policy.rego
For Gatekeeper or Kyverno, you would typically run a similar command using their respective tools.
Debugging Output
OPA Example:
JSON
{
  "result": false,
  "violations": [
    {
      "path": ["/spec/template/metadata/labels/app"],
      "message": "label 'app' must be 'MyApp'"
    }
  ]
}

#### Common Mistakes
and Follow-Ups
Typographical Errors: Ensure there are no typos in the label name or path.
Case Sensitivity: Check for case sensitivity issues, especially on systems where this matters (e.g., some Kubernetes implementations).
Policy Logic Issues: Review the policy logic to ensure it is correctly written and that all conditions are properly accounted for.

By following these steps, you can systematically debug why a label might not be recognized as expected by your OPA/Gatekeeper policy. This approach ensures thoroughness and helps in identifying any subtle issues that could cause validation failures.

