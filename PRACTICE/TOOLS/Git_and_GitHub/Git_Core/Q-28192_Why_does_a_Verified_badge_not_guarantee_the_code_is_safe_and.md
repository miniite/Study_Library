---
type: "practice"
difficulty: "Intermediate"
---

# Why does a 'Verified' badge not guarantee the code is safe, and what does it actually attest to?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Signed Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Signed_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with version control systems like Git, particularly on platforms like GitHub, a 'Verified' badge might seem reassuring at first glance. However, it’s crucial to understand that this badge does not guarantee the safety or correctness of the codebase. Instead, it attests to a higher level of security and trustworthiness regarding how the repository is accessed.

To break down the question step by step:

What the 'Verified' Badge Actually Atteststo: The 'Verified' badge in GitHub indicates that a repository has been checked for security vulnerabilities using tools provided by GitHub Security Advisories. This means that GitHub scans your repository against known vulnerability databases to ensure that no insecure dependencies are being used.

Why It Does Not Guarantee Code Safety: Even with this verification, code safety cannot be guaranteed solely based on the presence of a 'Verified' badge. There are several reasons why:

Tool Limitations: The tools used by GitHub Security Advisories can only detect certain types of vulnerabilities and may miss others. For example, they might not catch logical errors or design flaws.

False Positives/Negatives: Tools can sometimes generate false positives (flagging something as a vulnerability when it is not) or false negatives (missing actual vulnerabilities). This means that even with verification, the code could still contain real security issues.

Dynamic Security Issues: Security threats are dynamic and can change over time. A repository might be verified today but introduced new vulnerabilities tomorrow.

Additional Considerations:

Code Reviews: Regular code reviews by other developers or automated linters can help catch more subtle issues that static analysis tools might miss.
Continuous Integration/Deployment (CI/CD): Implementing a robust CI/CD pipeline can ensure that the code is tested thoroughly before being deployed. This includes running security scans as part of the deployment process.
Security Best Practices: Following established security best practices, such as using secure coding techniques, keeping dependencies up to date, and conducting regular audits, are essential for maintaining a safe codebase.

