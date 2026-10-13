---
type: "practice"
difficulty: "Expert"
---

# How would you design the org's approach to biometric data (face, fingerprint, gait) in ML given it's often irrevocable and specially regulated — what's categorically off-limits?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Privacy by Default](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/Privacy_by_Default.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In designing an organization’s approach to handling biometric data like face, fingerprints, and gait patterns in machine learning (ML), we must ensure strict adherence to legal and ethical standards while balancing operational efficiency and technological innovation. Biometric data is inherently sensitive and often irrevocable once used, making it subject to stringent regulations such as GDPR, HIPAA, and local laws. This solution will focus on categorically identifying what's off-limits in terms of collecting, storing, and processing biometric data.

#### Triaging the Question

The primary goal here is to define a comprehensive framework for managing biometric data that aligns with legal requirements while minimizing risks. The key aspects include:

Legal Compliance: Ensuring all practices comply with relevant laws.
Data Minimization: Collecting only necessary data and retaining it as minimally as possible.
Security and Anonymization: Protecting the integrity of biometric data during storage and processing.
Transparency and Consent: Managing informed consent from individuals.

#### Assumptions
The organization operates in multiple jurisdictions with differing regulations.
The ML model will be used for various applications, such as access control or identity verification.
Data is collected via third-party APIs or directly through end-user devices.

#### First Principles Solution

Data Collection

Category: Off-limits
Categorical Use: Biometric data should not be collected unless absolutely necessary and explicitly consented to by the individual.
Third-Party Data: Do not rely on third-party APIs that store or process biometric data, as this can introduce compliance risks.

Data Storage

Category: Off-limits
Ingestion: Biometric data should never be stored directly in databases. Instead, use secure hashing techniques to create anonymized identifiers.
Storage Location: Data must be stored in encrypted form with access controls enforced at all times.

Data Processing and ML Models

Category: Off-limits
Model Training: ML models should not process biometric data directly for training. Use proxy features or other anonymized representations.
Inference: Biometric data used in real-time inference must be handled securely, with strict controls on who can access it and under what conditions.

Data Retention

Category: Off-limits
Retention Periods: Define clear retention periods based on legal requirements, but aim to retain as little data as possible.
Erasure Obligation: Develop a process for securely erasing biometric data when it is no longer needed.

Transparency and Consent

Category: Off-limits

#### Cons
ent Management: Maintain a transparent consent management system where individuals can control their data and revoke consent at any time.
Privacy Impact Assessments (PIAs): Conduct regular PIAs to assess the impact of biometric data usage on privacy.

#### Closing Summary

By categorically defining what is off-limits in handling biometric data, we ensure that all operations adhere to stringent legal requirements and ethical standards. This approach not only mitigates compliance risks but also builds trust with stakeholders by demonstrating a commitment to privacy and security.

#### Common Mistakes/Follow-Ups
Mistake: Collecting biometric data without explicit consent.

#### Follow-Up
Implement strong consent management systems and regularly audit compliance.
Mistake: Storing raw biometric data directly in databases.

#### Follow-Up
Use secure hashing techniques for anonymization before storage.
Mistake: Using ML models to process biometric data directly.

#### Follow-Up
Develop proxy features or use third-party services that respect privacy.

**Comparison:**

```
Category	Off-limits
Data Collection	Do not collect biometric data unless explicitly consented to.
Storage	Store only anonymized identifiers, with strict access controls and encryption.
Processing	Do not process raw biometric data in ML models; use proxy features instead.
Retention	Define clear retention periods based on legal requirements but retain as little as possible.
Consent Management	Provide transparent mechanisms for users to control and revoke consent.
```

**Code:**

```python
# Example of using a secure hash function for anonymization
import hashlib

def anonymize_biometric_data(data):
    return hashlib.sha256(data.encode()).hexdigest()
```

**Example input / output:**

```text
Input:
- Raw biometric data from a user (e.g., face image, fingerprint)

Output:
- Anonymized identifier (hash) stored securely

Example Dry-Run:
- Raw Face Data: "John Doe's Face Image"
- Anonymized Identifier: "hash1234567890abcdef"

Input:
- Biometric data for inference in a ML model

Output:
- Proxy feature used for inference

Example Dry-Run:
- Raw Fingerprint Data: "John Doe's Fingerprint"
- Proxy Feature: "hashed_fingerprint_proxy_1234567890abcde"
```

**Data:**

```json
{
  "individual": {
    "name": "John Doe",
    "biometricData": {
      "face": "hash1234567890abcdef",
      "fingerprint": "hash234567890abcdefgh"
    },
    "consent": true,
    "retentionPeriod": "6 months"
  }
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Yes
6. No
7. Data Collection
8. Is Consent Given?
9. Anonymize Data
10. Store Raw Data?
11. Log and Erase
12. Encrypt and Store
13. Access Controls
14. Use Biometric Data?
15. Proxy Feature
16. Logging Only

