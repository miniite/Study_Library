---
type: "practice"
difficulty: "Expert"
---

# You inherit five CI systems from acquisitions with incompatible security models; design the multi-year consolidation so you converge security posture first (secrets, provenance, runner isolation) before converging tooling, and defend that ordering.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Supply Chain Security and Provenance](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/Supply_Chain_Security_and_Provenance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To consolidate five CI systems from acquisitions with incompatible security models, we need to design a multi-year strategy that prioritizes improving the security posture first (secrets management, provenance tracking, runner isolation) before standardizing on tooling across the board. This approach ensures a secure baseline and reduces the risk of introducing vulnerabilities during the transition.

The key steps are as follows:

Assess Current Security Postures: Begin by thoroughly understanding the current security models in each system.
Identify Common Vulnerabilities: Determine common weaknesses that need to be addressed, such as lack of secrets management or runner isolation.
Define a Baseline Security Model: Create a new security model that can encompass all existing systems while providing a secure foundation.
Implement Secret Management: Gradually migrate secrets from each system to a centralized or managed secret store.
Enhance Provenance Tracking: Implement mechanisms to track the history and lineage of builds, ensuring traceability.
Improve Runner Isolation: Ensure that runners are isolated both physically and logically to prevent compromise.
Standardize on Tooling: Once security is consolidated, focus on converging tooling practices.

This approach ensures a secure transition by addressing critical security gaps first, reducing the risk of vulnerabilities spreading across the systems.

**Comparison:**

```
System	Current State (Security)	Target State (Security)
A	Secrets: Yes, Provenance: No, Isolation: No	Secrets: Centralized, Provenance: Tracked, Isolation: Isolated
B	Secrets: Yes, Provenance: Yes, Isolation: No	Secrets: Centralized, Provenance: Tracked, Isolation: Isolated
C	Secrets: No, Provenance: Yes, Isolation: Yes	Secrets: Centralized, Provenance: Tracked, Isolation: Isolated
D	Secrets: Yes, Provenance: No, Isolation: Yes	Secrets: Centralized, Provenance: Tracked, Isolation: Isolated
E	Secrets: No, Provenance: No, Isolation: No	Secrets: Centralized, Provenance: Tracked, Isolation: Isolated
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[Assess Current Security Postures] --> B{Identify Common Vulnerabilities}
    B --> C(Define Baseline Security Model)
    C --> D1[Implement Secret Management]
    C --> D2[Enhance Provenance Tracking]
    C --> D3[Improve Runner Isolation]
    D1 --> E1[Migrate Secrets from System A]
    D1 --> E2[Migrate Secrets from System B]
    D2 --> F1[Enable Provenance for System A]
    D2 --> F2[Enable Provenance for System B]
    D3 --> G1[Isolate Runners in System A]
    D3 --> G2[Isolate Runners in System B]
```

**Example input / output:**

```text
Input:
- Existing CI systems with different security models

Output:
1. Baseline Security Model defined and adopted
2. Secrets from all systems migrated to a centralized store
3. Provenance tracking enabled for relevant systems
4. Runners isolated in all systems
5. Converged tooling practices implemented over time
```

**Data:**

```yaml
security_models:
  system_a: { secrets: true, provenance: false, isolation: false }
  system_b: { secrets: true, provenance: true, isolation: false }
  system_c: { secrets: false, provenance: true, isolation: true }
  system_d: { secrets: true, provenance: false, isolation: true }
  system_e: { secrets: false, provenance: false, isolation: false }
```

**Flow:**

1. Assess Current SecurityPostures
2. Identify CommonVulnerabilities
3. Define Baseline Security Model
4. Implement SecretManagement
5. Migrate Secrets from System A
6. Migrate Secrets from System B
7. Enhance Provenance Tracking
8. Enable Provenance for SystemA
9. Enable Provenance for SystemB
10. Improve Runner Isolation
11. Isolate Runners in System A
12. Isolate Runners in System B

