---
type: "practice"
difficulty: "Expert"
---

# A principal probe on your SLSA-level-3 mandate across 300 pipelines with a two-quarter deadline: where do you put the enforcement point, and how do you handle teams who genuinely can't comply in time without either blocking them or gutting the guarantee?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Supply Chain Security and Provenance](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/Supply_Chain_Security_and_Provenance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this principal probe on SLSA-level-3 mandate across 300 pipelines within a two-quarter deadline, we need to carefully triage and implement enforcement points while ensuring that teams who are genuinely struggling can still meet the deadline without compromising the overall security posture. The key is to place enforcement points strategically and ensure clear communication and support for non-compliant teams.

First, let's clarify what SLSA-level-3 mandates: it requires secure software supply chain practices, including source code provenance, build processes, binary integrity, and transparency of security vulnerabilities. Our goal is to enforce these practices without causing undue delays or resource strain on the teams.

#### Triage
and Enforcement Points
Where Do You Place the Enforcement Point?
Source Code Provenance (SCPro): This should be enforced at the earliest stage possible, ideally in version control repositories. We need to ensure that all code commits are signed with immutable keys, and provenance information is stored securely.
Build Integrity: Build processes must be validated before deployment. This can be enforced via CI/CD pipelines using security tools like SLSA v3 build policies. However, given the tight deadline, we may need to start enforcing this step after SCPro but as early as possible in the pipeline.
Binary Integrity and Transparency of Vulnerabilities: These aspects are harder to enforce without significant changes to existing systems. For now, we can focus on ensuring that binaries are signed and provenance information is stored securely, with a plan to address transparency later.
Handling Non-Compliant Teams
Enforcing Compliance Without Blocking
Assessment Phase: Conduct a thorough assessment of each pipeline's current state. Identify teams that are not compliant and determine the scope of work required.
Incremental Rollouts: Start by enforcing SCPro on a subset of pipelines, gradually rolling out build integrity checks over time. This allows us to identify any blockers early and address them.
Collaborative Solutions: Work closely with non-compliant teams to understand their constraints and offer tailored solutions. For instance, if a team lacks the necessary infrastructure, provide guidance or assist in setting up required tools.
Training and Support: Offer training sessions on SLSA best practices and provide ongoing support to help teams integrate new security measures into their workflows.
Avoiding Gutting the Guarantee
Critical Pipelines First: Prioritize critical pipelines that handle sensitive data or are part of core business functionality. Ensure these pipelines meet the highest compliance standards.
Risk-Based Approach: Use a risk-based approach to prioritize which teams and pipelines need immediate attention. Focus on those that pose the greatest security risks first.
Temporary Waivers: Grant temporary waivers for non-critical systems, allowing time for remediation without fully compromising the SLSA mandate. This approach requires careful tracking and reporting.

#### Closing Summary

By placing enforcement points strategically—starting with source code provenance and gradually moving to build integrity—we can ensure that we meet the SLSA-level-3 mandate while providing support to non-compliant teams. Clear communication, incremental rollouts, and collaborative solutions will help us achieve compliance without causing undue delays or resource strain.

#### Common Mistakes/Follow-Ups
Failing to Prioritize: Not prioritizing critical pipelines can lead to significant security vulnerabilities.
Overly Strict Enforcements: Focusing too heavily on all teams at once can cause bottlenecks and hinder progress.
Lack of Collaboration: Insufficient support for non-compliant teams can result in resistance and delays.

**Comparison:**

```
Action	Impact on Compliance	Impact on Team Performance
SCPro Enforcement	High	Low to Moderate
Build Integrity Checks	Medium	Low
```

**Code:**

```python
# Example Python script for automated SCPro enforcement
def enforce_sc_pro(pipeline):
    if not pipeline.has_valid_code_signatures():
        raise Exception("Code signatures missing or invalid")
    if not pipeline.provenance_information_securely_stored():
        raise Exception("Provenance information not securely stored")
```

**Example input / output:**

| Pipeline Name | Initial Status | Required Actions |
|---------------|----------------|-----------------|
| CriticalApp    | Non-compliant   | SCPro, Build Integrity |
| CustomerPortal | Compliant       | -                |

**Data:**

```json
{
  "pipelines": [
    {
      "name": "CriticalApp",
      "status": "Non-compliant",
      "required_actions": ["SCPro enforcement", "Build integrity checks"]
    },
    {
      "name": "CustomerPortal",
      "status": "Compliant",
      "required_actions": []
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Initial Pipeline Assessment
4. Is Critical?
5. Enforce SCPro
6. Provide Support and Training
7. Gradually Roll Out BuildIntegrity Checks
8. Collaborate on CustomSolutions
9. Monitor Compliance andPerformance
10. Maintain Communication andSupport

