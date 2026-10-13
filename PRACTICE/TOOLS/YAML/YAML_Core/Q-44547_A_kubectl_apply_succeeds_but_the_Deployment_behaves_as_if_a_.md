---
type: "practice"
difficulty: "Deep"
---

# A kubectl apply succeeds but the Deployment behaves as if a field was ignored. How would you confirm whether the field is unknown to the schema versus dropped by strategic-merge, given apply does not error on unknown YAML keys by default?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kustomize Patching](<../../../../TOOLS/YAML/YAML_Core/Kustomize_Patching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with Kubernetes Deployments and kubectl apply, it's common to encounter scenarios where a field in your YAML manifest is either unrecognized or modified due to strategic merge patching, even if apply does not error on unknown fields by default. To determine whether a specific field was ignored because it’s unknown to the schema or dropped by strategic-merge, you need to methodically inspect and compare the current state of the Deployment with your intended YAML configuration.

#### Triaging the Question

The core issue here is understanding why a particular field in your YAML manifest appears to be missing after running kubectl apply. The question probes your knowledge of how Kubernetes handles unknown fields and strategic merge patches, as well as your ability to trace changes accurately. Key assumptions include:

You have a Deployment managed by Kubernetes.
A kubectl apply command was issued with new or modified YAML files.
The resulting state of the Deployment does not match expectations.

#### First Principles Solution

Identify the Missing Field:

Confirm which field is missing from your current Deployment's configuration.

Check Your Original YAML Files:

Compare the original (intended) YAML file with the updated one.
Look for any unexpected differences that might indicate strategic merge modifications.

Use kubectl diff and kubectl patch:

Utilize kubectl diff to identify what changes have been applied or rejected.
Employ kubectl patch with a JSON Patch to test specific fields and observe the behavior.

Inspect the Deployment's Status:

Check the status field in the Deployment’s Kubernetes object to see if it reflects any unapplied configurations.

Verify the Manifest Version:

Ensure that your YAML manifest is using a compatible schema version with the cluster API.
Step-by-Step Solution

Identify the Missing Field:

SH
kubectl get deployment <deployment-name> -o yaml > current.yaml

Compare current.yaml with your intended manifest to find discrepancies.

Use kubectl diff:

This tool compares two YAML files and outputs differences.
SH
kubectl diff -f original.yaml -f current.yaml

This will show you what has changed or is missing in the current state compared to the original.

Inspect the Deployment’s Status:

SH
kubectl get deployment <deployment-name> -o json > deployment-status.json

Look for the status field, which might contain information about unapplied configurations.

JSON
"status": {
  ...
  "observedGeneration": 1,
  "replicas": 3,
  "updatedReplicas": 3,
  "availableReplicas": 3,
  "unavailableReplicas": 0,
  "conditions": [
    ...
  ]
}

Test with kubectl patch:

Create a JSON Patch to test the field in question.
JSON
{
  "op": "add",
  "path": "/spec/template/spec/containers/0/env/-",
  "value": {
    "name": "TEST_ENV_VAR",
    "value": "test-value"
  }
}

Apply this patch to the Deployment and observe if it takes effect.

Verify Schema Compatibility:

Ensure that your YAML manifest uses a compatible schema version.
SH
kubectl explain deployment.spec.template.spec.containers

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake: Assuming all unknown fields are ignored without checking the strategic merge behavior.

#### Follow-Up
If kubectl diff shows a field is missing, but kubectl patch does not apply it, this likely indicates strategic merge has dropped it due to schema changes.

By methodically following these steps, you can accurately determine whether a field was ignored because it’s unknown to the schema or simply dropped by Kubernetes' strategic-merge mechanism.

**Code:**

```
SH
Copy
kubectl diff -f original.yaml -f current.yaml
```

