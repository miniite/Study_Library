---
type: "practice"
difficulty: "Expert"
---

# Your container scan fails the build on a critical CVE in a base image you can't immediately patch; how do you decide between blocking, exempting with an expiry, or swapping base images?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Flaky Tests](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Flaky_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When facing a situation where your CI/CD pipeline fails because of a critical CVE in a base image that can't be patched immediately, you need to carefully consider the trade-offs between blocking the build, exempting it with an expiry, or swapping the base images. The decision involves understanding the risk profile and balancing it against the cost and impact on the project timeline.

Firstly, let's clarify what the question is probing:

Risk Management: How you assess and mitigate security risks.
Operational Flexibility: Balancing between strict policies and pragmatic workarounds.
Technical Decisions: Evaluating different technical solutions and their impacts.

#### Assumptions
You have a CI/CD pipeline that automatically runs container scans using tools like Clair, Aqua Security, or Trivy.
The failing CVE is critical (e.g., high severity) but does not affect the functional requirements of your application.
The base image in question is widely used and has many dependencies.

#### First Principles Solution
1. Blocking the Build

#### Pros
Ensures no insecure images are deployed to production.
Maintains a high security baseline across all deployments.

#### Cons
Delays your development cycle, as you must wait for patches or fixes.
Could block downstream dependencies that rely on the failed base image.
2. Exempting with Expiry

#### Pros
Allows the build to continue in the short term while mitigating immediate risk.
Enables teams to work around urgent deadlines without compromising security posture.

#### Cons
Requires a well-defined and enforced policy for exemption management.
There is a risk of indefinite exemptions, leading to increased long-term maintenance costs.
3. Swapping Base Images

#### Pros
Faster resolution as you can use an alternative base image that has the necessary patches.
Potentially reduces the number of downstream issues by quickly fixing the problem source.

#### Cons
May introduce additional dependencies or compatibility issues in your application.
Requires careful validation to ensure the new base image meets all requirements.
Decision Factors
Risk Severity: How critical is the CVE? Is it exploitable under real-world conditions?
Impact on Deployment: What are the immediate and long-term implications of delaying deployment versus allowing a temporary exception?
Alternatives Available: Are there suitable alternative base images that can be used without significant changes to your application?
Step-by-Step Reasoning

Assess Severity:

Review the CVE details and determine if it poses an immediate threat.

#### Cons
ult security advisories and community feedback for guidance.

Impact Analysis:

Evaluate the impact on current deployments and future ones.

#### Cons
ider how this issue might affect other projects or services that rely on the same base image.

Stakeholder Engagement:

Involve product managers, security teams, and development leads to gather input.
Document decision-making criteria and rationale for transparency.

Implement Strategy:

Choose blocking if the risk is too high and no immediate alternative exists.
Use exemptions with a defined expiry period if acceptable risk and a patch is expected soon.
Swap base images if available, ensuring thorough testing before deployment.

#### Closing Summary

In summary, when deciding between blocking, exempting, or swapping base images in response to a critical CVE, you should carefully weigh the security, operational, and technical implications. A pragmatic approach that balances risk management with project timelines is key.

#### Common Mistakes
Ignoring the long-term consequences of exemptions.
Failing to properly document and manage exceptions.
Overlooking the availability of alternative base images when they might provide a faster solution.

#### Follow-Ups
Monitoring: Continuously monitor the affected services for any unusual behavior post-deployment.
Feedback Loops: Regularly update your CI/CD pipelines with new security insights to improve future decision-making.
Automated Patching: Implement automated patching mechanisms or strategies to expedite the remediation process.

**Comparison:**

```
Decision	Pros	Cons
Blocking	Ensures high security baseline.	Delays deployment; blocks downstream dependencies.
Exempting	Allows urgent development while mitigating risk.	Requires strict policy enforcement; can lead to indefinite exemptions.
Swapping Base Images	Faster resolution with alternative base images.	May introduce new dependencies or compatibility issues.
```

**Code:**

```python
# Example code snippet for handling CVE exemptions in a CI/CD pipeline

def handle_cve_exemption(cve, build_status):
    if is_critical_cve(cve) and not can_patch_immediately(build_status.base_image):
        return {"exempted": True, "expiry_date": datetime.now() + timedelta(days=7)}
    else:
        return {"exempted": False, "reason": "patch available"}

def is_critical_cve(cve_details):
    # Check CVE severity and impact
    pass

def can_patch_immediately(base_image):
    # Check if a patch for the critical CVE exists in the base image repository
    pass
```

**Example input / output:**

```text
input_data = {
  "cve": {
    "id": "CVE-2021-44228",
    "severity": "CRITICAL",
    "affected_versions": ["1.16.0", "1.17.0"],
    "patch_available": false,
    "exploitability": 9
  },
  "build_status": {
    "base_image": "alpine:3.15",
    "patches_applied": []
  }
}

output = handle_cve_exemption(input_data["cve"], input_data["build_status"])
print(output) # {'exempted': True, 'expiry_date': datetime(2024, 6, 1)}
```

**Data:**

```json
{
  "cve": {
    "id": "CVE-2021-44228",
    "severity": "CRITICAL",
    "affected_versions": ["1.16.0", "1.17.0"],
    "patch_available": false,
    "exploitability": 9
  },
  "build_status": {
    "base_image": "alpine:3.15",
    "patches_applied": []
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Assess CVE Severity
6. Is the CVE critical?
7. Exempt with Expiry (if possible)
8. Block Build
9. Can we patch immediately?
10. Patch and Deploy
11. Swap Base Images (if available)
12. Document Changes
13. Deploy with New Image

