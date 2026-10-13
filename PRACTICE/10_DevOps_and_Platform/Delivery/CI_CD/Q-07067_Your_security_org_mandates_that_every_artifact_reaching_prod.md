---
type: "practice"
difficulty: "Expert"
---

# Your security org mandates that every artifact reaching prod carry SLSA level-3 provenance; you have 300 pipelines and a two-quarter deadline. Lay out the rollout, the enforcement point, and how you handle teams who can't comply in time.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Supply Chain Security and Provenance](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/Supply_Chain_Security_and_Provenance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that all artifacts reaching production carry SLSA level-3 provenance within a two-quarter deadline, we need to establish a comprehensive CI/CD rollout plan. The key is to identify an enforcement point where compliance can be rigorously checked, and then ensure all teams adhere to this standard through education and tooling support.

Firstly, let's outline the strategy:

Enforcement Point: We will enforce SLSA level-3 provenance checks at our deployment gate in Jenkins X or a similar CI/CD platform.
Rollout Plan: This involves a phased approach to upgrade existing pipelines, ensuring that all necessary metadata is captured and verified.
Support & Training: Provide tools, training, and support for teams to meet the new requirements.

Now, let's break down each step in detail:

#### Triaging the Question

The core of this question revolves around:

Ensuring CI/CD pipelines produce artifacts with SLSA level-3 provenance.
Defining an enforcement point within the pipeline lifecycle.
Handling non-compliant teams to ensure compliance by the deadline.

#### Assumptions

We are using a CI/CD platform like Jenkins X, which provides hooks for enforcing security policies.
The security org has defined clear criteria and standards for SLSA level-3 provenance.
Each pipeline is associated with a specific project or service.
Teams have access to the necessary tools and training.

#### First-Principles Solution

Identify Enforcement Points:

Define an enforcement point in our CI/CD pipeline lifecycle where we can reliably verify SLSA level-3 provenance. This is typically at the "deploy" stage, just before artifacts are pushed to production.

Rollout Plan:

Phase 1: Audit Existing Pipelines: Begin by auditing existing pipelines to identify gaps in provenance metadata.
YAML
pipeline:
  steps:
    - name: build
      image: gcr.io/our-namespace/build-image:latest
      command: ['make', 'build']
    - name: verify-provenance
      image: gcr.io/our-namespace/provenance-verifier:latest
      command: ['verify', '--output=provenance.json']
Phase 2: Upgrade Pipelines: Gradually upgrade existing pipelines to capture and validate SLSA metadata.
YAML
pipeline:
  steps:
    - name: build
      image: gcr.io/our-namespace/build-image:latest
      command: ['make', 'build']
    - name: verify-provenance
      image: gcr.io/our-namespace/provenance-verifier:latest
      command: ['verify', '--output=provenance.json']
Phase 3: Continuous Monitoring: Implement continuous monitoring to ensure all pipelines are compliant and report any deviations.
YAML
pipeline:
  steps:
    - name: build
      image: gcr.io/our-namespace/build-image:latest
      command: ['make', 'build']
    - name: verify-provenance
      image: gcr.io/our-namespace/provenance-verifier:latest
      command: ['verify', '--output=provenance.json']

Support & Training:

Provide comprehensive training sessions for teams to understand the importance of SLSA level-3 provenance and how to implement it.
Offer documentation, templates, and best practices.
Set up a helpdesk or support channel for teams with questions.

#### Closing Summary

By enforcing SLSA level-3 provenance checks at our deployment gate and implementing a phased rollout plan, we can ensure that all artifacts reaching production meet the required security standards. Continuous monitoring will help identify any gaps and ensure compliance. Providing robust support and training will help mitigate resistance from teams who may face challenges in meeting the deadline.

#### Common Mistakes/Follow-Ups

Common Mistake: Overlooking existing pipelines during the audit phase, leading to missed deadlines.

#### Follow-Up
Ensure all pipelines are audited and upgraded to avoid any gaps in compliance.

Common Mistake: Insufficient support and training for teams, leading to delays or non-compliance.

#### Follow-Up
Provide ample resources and ongoing support to ensure smooth adoption.

**Comparison:**

```
Phase	Actions
Audit	Review existing pipelines, identify gaps
Upgrade	Modify existing pipeline configurations to include provenance checks
Monitor	Implement continuous monitoring for compliance
```

**Code:**

```yaml
# Example of a pipeline step with provenance verification
pipeline:
  steps:
    - name: build
      image: gcr.io/our-namespace/build-image:latest
      command: ['make', 'build']
    - name: verify-provenance
      image: gcr.io/our-namespace/provenance-verifier:latest
      command: ['verify', '--output=provenance.json']
```

**Example input / output:**

```text
Input:
Pipeline configuration with a build step and a provenance verification step.

Output:
Successfully built the application. Provenance verified and recorded in 'provenance.json'.
```

**Data:**

```json
{
  "pipelines": [
    {
      "name": "example-pipeline",
      "steps": [
        { "name": "build", "image": "gcr.io/our-namespace/build-image:latest" },
        { "name": "verify-provenance", "image": "gcr.io/our-namespace/provenance-verifier:latest" }
      ]
    }
  ]
}
```

**Flow:**

1. Start
2. Audit Existing Pipelines
3. Identify Gaps
4. Document and Report Gaps
5. Begin Upgrading Pipelines
6. Monitor ComplianceContinuously
7. Ensure Full Compliance byDeadline
8. Provide Training and Support

