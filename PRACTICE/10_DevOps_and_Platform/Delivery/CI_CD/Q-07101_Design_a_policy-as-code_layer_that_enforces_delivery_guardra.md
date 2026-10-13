---
type: "practice"
difficulty: "Expert"
---

# Design a policy-as-code layer that enforces delivery guardrails (signed artifacts, required checks, protected environments, approved base images) uniformly across every pipeline regardless of which CI tool a team uses; where does enforcement live and how do you avoid it being a bypassed rubber stamp?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Golden Pipelines and Templates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Golden_Pipelines_and_Templates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a policy-as-code layer to enforce delivery guardrails, our goal is to create a robust and reliable system that ensures consistency across all pipelines, independent of the CI tool used by each team. This involves several key components: defining policies, enforcing them, and ensuring they are consistently applied without being bypassed or treated as mere rubber stamps.

The core idea here is to leverage policy-as-code (PAC) to centralize and standardize guardrails such as signed artifacts, required checks, protected environments, and approved base images. We need to ensure that these policies are automatically enforced at the point of deployment in each pipeline, and we must design a mechanism that prevents bypassing or overriding these rules easily.

**Code:**

```yaml
# Example policy-as-code YAML
policies:
  - name: SignedArtifactsPolicy
    enforce: True
    checks:
      - artifactSignatureCheck

  - name: RequiredChecksPolicy
    enforce: True
    checks:
      - staticCodeAnalysis
      - unitTests
      - securityScanning

  - name: ProtectedEnvironmentsPolicy
    enforce: True
    environments:
      - prod

  - name: ApprovedBaseImagesPolicy
    enforce: True
    images:
      - approvedImageRepo/*

# Example policy enforcement YAML
pipelineConfigurations:
  pipelineName: examplePipeline
  policiesToApply:
    - SignedArtifactsPolicy
    - RequiredChecksPolicy
    - ProtectedEnvironmentsPolicy
    - ApprovedBaseImagesPolicy
```

**Example input / output:**

```

This is demonstrated in the YAML snippets provided, where we define various policies and configure pipelines to apply them. Each policy can have its own set of checks and environments, allowing us to enforce a wide range of guardrails uniformly across different CI tools.

YAML
Copy
# Example input (pipeline configuration)
pipelineName: examplePipeline
policiesToApply:
  - name: SignedArtifactsPolicy
    enforce: True
    checks:
      - artifactSignatureCheck

  - name: RequiredChecksPolicy
    enforce: True
    checks:
      - staticCodeAnalysis
      - unitTests
      - securityScanning

# Example output (policy enforcement)
Pipeline examplePipeline will execute the following policies:
- SignedArtifactsPolicy (checks: artifactSignatureCheck) enforced.
- RequiredChecksPolicy (checks: staticCodeAnalysis, unitTests, securityScanning) enforced.
```

**Flow:**

1. Yes
2. No
3. Pipeline Configuration
4. Apply Policies?
5. Enforce SignedArtifactsPolicy
6. Enforce RequiredChecksPolicy
7. EnforceProtectedEnvironmentsPolicy
8. EnforceApprovedBaseImagesPolicy
9. Deployment
10. Pipeline Proceeds WithoutEnforcement

