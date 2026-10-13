---
type: "practice"
difficulty: "Deep"
---

# How do you elicit a security scenario for your platform's software supply chain ('a malicious base image or transitive dependency reaches production')? What sensitivity points around SLSA provenance, admission policy, and signing does it expose across the pipeline?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scenario Prioritization](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Scenario_Prioritization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To elicit a security scenario for our platform's software supply chain—specifically focusing on a malicious base image or transitive dependency reaching production—we need to consider various sensitivity points related to SLSA (Software bill of Materials, Provenance, and Integrity) standards. This involves understanding the pipeline stages where vulnerabilities can be introduced and ensuring robust mechanisms are in place for provenance, admission policy, and signing.

#### Triage

The question is probing our ability to identify potential security weaknesses within the software supply chain, particularly focusing on how a malicious component might infiltrate production through the build process. It aims to assess our understanding of SLSA practices and our capability to design and implement effective mitigation strategies.

#### First-Principles Solution

Provenance Management:

Define clear policies for source code repositories, ensuring that all dependencies are from known, trusted sources.
Implement automated checks for versioning and integrity verification using tools like gpg or sigstore.

Admission Policy:

Establish strict admission criteria for base images and dependencies before they enter the build pipeline.
Use container image scanning tools to detect any anomalies in the base image or transitive dependencies.

Signing Mechanisms:

Ensure all artifacts are signed using a trusted key, which can be verified by the CI/CD system.
Utilize notary services like sigstore for secure artifact signing and verification.
Detailed Breakdown

Provenance Management:

Source Code Repositories: Verify that dependencies in repositories (e.g., Git) are from trusted sources, possibly through code reviews or automated scanning tools.
Dependency Integrity Verification: Use gpg signatures or sigstore to ensure the integrity of downloaded dependencies. For example:
BASH
wget https://example.com/package.tar.gz
gpg --verify package.tar.gz.sig
tar -xvf package.tar.gz
Tooling Integration: Integrate dependency scanning tools into CI/CD pipelines, such as Dependabot, which can automatically update and verify dependencies.

Admission Policy:

Base Image Validation: Use container image scanning services (e.g., anchore, quay.io) to scan the base images for known vulnerabilities.
Transitive Dependency Scanning: Implement policies to ban or restrict certain dependencies based on their provenance and known risks. For example, if a dependency has a history of security issues, it should be disallowed in the pipeline.

Signing Mechanisms:

Artifact Signing: Sign all artifacts with a trusted key before deployment, ensuring that only signed artifacts can proceed to production.
Notary Services: Use sigstore or similar services to manage artifact signatures and ensure they are verified during deployment:
BASH
notary login --email example@org.com --server https://notary.example.com
notary sign --target my-app-1.0.0.tar.gz --key key.pem

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common mistakes
include relying solely on manual processes or failing to integrate security checks into the CI/CD pipeline. It’s crucial to have automated and continuous verification mechanisms in place.

Potential follow-ups might involve discussing specific tools used, how to handle false positives/negatives in vulnerability scanning, and strategies for managing vendor relationships and ensuring compliance with industry standards like SLSA 3.0.

**Comparison:**

```
Component	Verification Method
Source Code	Manual code review + automated
Base Image	Container image scanning
Dependency	Dependency versioning + integrity
Artifacts	Signing with trusted keys
```

**Code:**

```bash
# Example of using gpg to verify a package's signature
wget https://example.com/package.tar.gz
gpg --verify package.tar.gz.sig

# Example of using notary for artifact signing and verification
notary login --email example@org.com --server https://notary.example.com
notary sign --target my-app-1.0.0.tar.gz --key key.pem
```

**Example input / output:**

```text
# Example input for dependency verification in CI/CD pipeline
Input:
- package.tar.gz (unverified)
- package.tar.gz.sig (signature file)

Output:
- True (if verified) or False (if not verified)
```

**Data:**

```json
{
  "provenance_policy": {
    "allowed_sources": ["trusted-repo.org", "company-gitlab.com"],
    "disallowed_dependencies": ["unmaintained-lib", "old-version"]
  },
  "admission_criteria": {
    "image_scan_required": true,
    "dependency_scanning_required": true
  },
  "signing_keys": [
    { "id": "key1.pem", "type": "RSA" },
    { "id": "key2.pem", "type": "ECDSA" }
  ]
}
```

**Flow:**

1. Source Code
2. Verify Source CodeRepositories
3. Base Image
4. Scan Base Images forVulnerabilities
5. Dependencies
6. Verify Dependency Integrity
7. Artifacts
8. Sign Artifacts with Trusted Keys
9. Audit and Monitoring
10. Continuous Verification
11. Trusted Source
12. Untrusted Source
13. No Vulnerabilities
14. Vulnerabilities Found
15. Intact
16. Damaged Integrity
17. Signed
18. Unsigned
19. Pass
20. Fails

