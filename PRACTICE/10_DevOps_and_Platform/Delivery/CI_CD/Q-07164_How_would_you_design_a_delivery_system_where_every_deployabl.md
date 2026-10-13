---
type: "practice"
difficulty: "Expert"
---

# How would you design a delivery system where every deployable artifact carries a cryptographically verifiable graph of exactly which source, dependencies, and build steps produced it, and why does that graph matter more than a single signature?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Test Data Governance](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/Test_Data_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a delivery system where each deployable artifact is accompanied by a cryptographically verifiable graph of its origins, dependencies, and build steps, we need to ensure transparency, traceability, and integrity in our software development pipeline. This approach is crucial for maintaining trust in the software supply chain, especially as vulnerabilities like those seen with the SolarWinds attack highlight the importance of secure artifact verification.

At a high level, this system involves several key components:

Source Code Management: Tracking the precise versions of source code that contribute to an artifact.
Dependency Management: Ensuring all dependencies are correctly identified and tracked.
Build Steps: Recording every build step, including tools used, configurations, and environment details.
Verification Mechanism: Using cryptographic hashes or signatures to verify the integrity of each component.

Let's break this down from first principles.

#### Triage

The question is asking how to design a CI/CD system where deployable artifacts include detailed provenance information that can be cryptographically verified. This involves understanding and documenting every step of the build process, including source code, dependencies, and tools used. The core idea is to create a verifiable graph (a Directed Acyclic Graph or DAG) representing the entire build history.

#### Assumptions
We are working in an environment with modern CI/CD tools like Jenkins, GitLab CI, or GitHub Actions.
Each artifact must be uniquely identifiable and versioned.
Dependencies need to be explicitly tracked and their provenance verified.
Step-by-Step Solution

Source Code Management:

Use a VCS (Version Control System) like Git to track every commit and tag the source code version with unique identifiers.
Git Repository
Commit Hashes
Sources Versioned by CommitHash
Source Code Artifacts

Dependency Management:

Maintain a list of all dependencies, their versions, and sources.
Use tools like npm, pip, or Maven to handle dependencies, ensuring they are from trusted sources.
Source Code Artifacts
Dependencies List
Dependency Versions
Traffic Light for Dependencies

Build Steps:

Document every build step, including tool versions, configurations, and environment details.
Use scripts to automate the build process and capture logs for verification.
Source Code Artifacts
Build Scripts
Environment Details
Tool Versions
Build Logs

Verification Mechanism:

Generate cryptographic hashes or signatures for each component of the artifact.
Store these hashes in a secure, immutable ledger like a blockchain or a centralized database.
Source Code Artifacts
Hashes/Signatures
Immutable Ledger
Verification Interface
Why Does the Graph Matter More Than a Single Signature?

A cryptographically verifiable graph provides several advantages over a single signature:

Traceability: The graph allows you to trace back every component of an artifact, ensuring that all parts are from trusted sources.
Transparency: It offers a clear record of the build process, making it easier to audit and understand how an artifact was created.
Reproducibility: With detailed records, anyone can reproduce the build environment and steps, ensuring consistent results.
Security: By verifying the entire build history, you can detect any tampering or unauthorized changes.

#### Closing Summary

In summary, designing a delivery system that cryptographically verifies the provenance of each artifact requires a comprehensive approach involving source code management, dependency tracking, detailed build records, and robust verification mechanisms. The graph matters more because it provides a complete, transparent, and verifiable history of the build process, offering much greater security and trust compared to a single signature.

#### Common Mistakes/Follow-Ups
Overlooking Dependencies: Ensuring all dependencies are tracked and verified is crucial.
Lack of Automation: Manual steps can introduce errors; automation should be fully integrated into the pipeline.
Data Integrity: Secure storage and transmission mechanisms for hashes and signatures must be in place.

**Comparison:**

```
Step	Artifact	Description
1	Source Code Artifacts	Versioned commits and files
2	Dependencies	Explicitly tracked, verified
3	Build Steps	Detailed record of every step
4	Hashes/Signatures	Cryptographically secure
```

**Code:**

```python
# Example Python script to generate hashes for source code artifacts
import hashlib

def get_hash(file_path):
    with open(file_path, "rb") as f:
        file_hash = hashlib.sha256()
        while chunk := f.read(8192):
            file_hash.update(chunk)
    return file_hash.hexdigest()

# Example usage
source_code_file = "/path/to/source/code"
artifact_hash = get_hash(source_code_file)
print("Hash:", artifact_hash)
```

**Example input / output:**

```text
Input:
- Source code files: /src/main.js, /lib/utils.js
- Dependencies list: npm@7.0.0, pip@20.2.4

Output:
{
  "commit": "abc123def456",
  "dependencies": [
    { "name": "npm", "version": "7.0.0" },
    { "name": "pip", "version": "20.2.4" }
  ],
  "build_steps": [
    { "tool": "make", "command": "make install", "output": "/path/to/installation" },
    { "tool": "npm", "command": "npm install -g <package>" }
  ],
  "hashes": {
    "source_code": "e3b0c44298fc... (main.js)",
    "dependencies": [
      "5f4dcc3b5aa765d61d83... (npm@7.0.0)",
      "7a4e7e8b3d34a... (pip@20.2.4)"
    ]
  }
}
```

**Data:**

```json
{
  "commit": "abc123def456",
  "dependencies": [
    { "name": "npm", "version": "7.0.0" },
    { "name": "pip", "version": "20.2.4" }
  ],
  "build_steps": [
    { "tool": "make", "command": "make install", "output": "/path/to/installation" },
    { "tool": "npm", "command": "npm install -g <package>" }
  ],
  "hashes": {
    "source_code": "e3b0c44298fc...",
    "dependencies": [
      "5f4dcc3b5aa765d61d83... (npm)",
      "7a4e7e8b3d34a... (pip)"
    ]
  }
}
```

**Flow:**

1. Source Code
2. Commit
3. Dependent Modules
4. Build Scripts
5. Build Artifacts
6. Hash Calculation
7. Immutable Ledger
8. Traffic Light for Dependencies
9. Environment Details
10. Tool Versions
11. Build Logs

