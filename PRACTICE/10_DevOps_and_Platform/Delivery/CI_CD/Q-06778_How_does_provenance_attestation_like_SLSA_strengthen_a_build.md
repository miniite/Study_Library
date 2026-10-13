---
type: "practice"
difficulty: "Expert"
---

# How does provenance/attestation (like SLSA) strengthen a build pipeline?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Supply Chain Security and Provenance](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/Supply_Chain_Security_and_Provenance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Provenance and attestation, such as those provided by Secure Software Lifecycles and Artifacts (SLSA), are crucial components in ensuring the integrity of software builds within CI/CD pipelines. By providing detailed records of how a piece of code was built, SLSA helps teams verify that the source is trusted and secure. This is especially important in complex, distributed environments where multiple parties might be involved in the build process.

In essence, SLSA strengthens a build pipeline by offering verifiable evidence about the steps taken during the build process. Let's break down how it works from first principles:

Understanding SLSA: SLSA is a framework that defines best practices for secure software supply chains. It includes four levels (SLSA 1, 2, 3, and 4) that incrementally increase the trust in the build process.

Key Concepts:

Provenance: This refers to the history of how a piece of code was built.
Attestation: This is a form of verification that confirms the provenance is correct.

Steps Involved:

Source Code Management (SCM): The source code must be stored securely and accessed via a trusted SCM system.
Build Tooling: The build process itself should be executed using tools and environments that are also secure and verifiable.
Supply Chain Transparency: Detailed records of the build steps, dependencies, and configurations must be kept.

Now, let's dive into how SLSA integrates with CI/CD pipelines:

Initialization:

Source Code Management: Ensure your source code is stored in a secure repository (e.g., GitHub Enterprise).
CI/CD Setup: Configure your CI pipeline to use these repositories and pull requests.

Provenance Generation:

During the build process, gather all relevant information: timestamps, versions of dependencies, build commands, etc.
Store this information in a way that can be later verified (e.g., via JSON or YAML files).

Attestation Process:

Use tools like SLSA to generate attestation data.
This could involve checking for the presence and correctness of required steps in the build process.

Verification:

In a production environment, these records can be verified against known good states.
Tools like SLSA verifiers can automatically check if all necessary steps were followed.

Here’s an example workflow:

YAML
# Example SLSA Provenance File (slsa.json)
{
  "version": 1,
  "hashes": [
    {
      "algo": "sha256",
      "digest": "..."
    }
  ],
  "steps": [
    {
      "id": "1",
      "name": "Checkout Source Code",
      "input": {},
      "output": {}
    },
    {
      "id": "2",
      "name": "Build Project",
      "input": {
        "files": ["./*.go"]
      },
      "output": {
        "artifacts": ["bin/example"]
      }
    }
  ]
}
Integration with CI/CD:
In your CI pipeline, ensure that SLSA steps are executed as part of the build process.
Use a tool like SlsaFed to generate and verify these artifacts.

In summary, SLSA significantly enhances the security and reliability of software builds by providing verifiable evidence about the build process. By implementing SLSA in your CI/CD pipelines, you can ensure that only trusted code reaches production.

**Code:**

```json
{
  "version": 1,
  "hashes": [
    {
      "algo": "sha256",
      "digest": "..."
    }
  ],
  "steps": [
    {
      "id": "1",
      "name": "Checkout Source Code",
      "input": {},
      "output": {}
    },
    {
      "id": "2",
      "name": "Build Project",
      "input": {
        "files": ["./*.go"]
      },
      "output": {
        "artifacts": ["bin/example"]
      }
    }
  ]
}
```

**Flow:**

1. Source Code Management
2. CI Pipeline Initiation
3. Provenance Generation
4. Attestation Check
5. Build Execution
6. Verification and Deployment

