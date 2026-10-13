---
type: "practice"
difficulty: "Deep"
---

# How do you architect PII handling differently for internal-employee-data models (HR, productivity) vs customer-data models given works-council and labor laws?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ PII Controls for Vendor Models](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/PII_Controls_for_Vendor_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing data privacy systems for AI models that handle personal information (PII), it's crucial to tailor your architecture based on the nature of the data and the legal requirements. For internal-employee-data models, like HR or productivity metrics, you typically need compliance with works-council agreements and labor laws. In contrast, customer-data models must comply with a broader range of regulations such as GDPR, CCPA, etc.

To architect PII handling differently for these two types of data, we need to consider the legal frameworks, data sensitivity, and potential consequences of mishandling. Let's break this down step by step from first principles.

**Comparison:**

```
Data Type	Compliance Requirements	Key Considerations
Employee	Works Council Agreements, Labor Laws	Anonymization, pseudonymization
Customer	GDPR, CCPA, Data Subject Rights	Full data masking, differential privacy
```

**Flow:**

1. Yes
2. No
3. Data Collection
4. Is Employee Data?
5. Anonymize and Pseudonymize
6. Mask Data Fully, ApplyDifferential Privacy
7. Storage with Access Controls
8. Storage with Detailed Logging
9. Compliance Auditing
10. Compliance Auditing
11. Audit Reports

