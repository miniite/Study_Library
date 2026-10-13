---
type: "practice"
difficulty: "Deep"
---

# How do you design the org's stance and technical guardrails on shadow AI — employees pasting customer PII into public chatbots — as an architectural problem not just policy?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Fine-tuned Models as Sensitive Artifacts](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/Fine-tuned_Models_as_Sensitive_Artifacts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Designing an organization’s stance on shadow AI, where employees might paste Customer Personally Identifiable Information (PII) into public chatbots, is a multifaceted challenge that requires both strategic thinking and robust technical solutions. The key idea here is to balance the need for transparency and user-friendliness with stringent data privacy controls. This problem transcends mere policy enforcement; it necessitates an architecture designed to detect, mitigate, and prevent unauthorized PII exposure.

#### Triage
What the Question Is Asking

The core of this question lies in understanding how to architect a system that can identify and handle PII effectively while preventing employees from inadvertently (or maliciously) exposing sensitive customer data. The interviewer is probing your ability to design technical guardrails, considering edge cases, and ensuring compliance with regulations such as GDPR or HIPAA.

Step-by-Step Solution

Identify Key Components:

Data Collection: How PII is collected in public chatbots.
Detection Mechanisms: Systems that can identify when PII is present.
Mitigation Strategies: Actions to take once PII is detected, such as redaction or blocking access.
User Education and Training: Ensuring employees understand the risks and protocols.

Technical Architecture:

Data Collection Mechanisms: Ensure all data processed by public chatbots is logged and monitored for patterns of PII exposure.
Real-time Detection Systems: Use machine learning models to identify common patterns of PII such as names, addresses, email IDs, etc.
Redaction Techniques: Implement automated redaction tools that can replace or mask sensitive information before it leaves the system.

Policy and Compliance:

Define clear policies around data handling and user behavior in public chatbots.
Regularly audit systems to ensure compliance with regulatory requirements.

User Education and Training:

Provide regular training sessions for employees on recognizing PII and understanding the risks associated with exposing it.
Implement a reporting mechanism where users can flag suspicious interactions or data leaks.

#### Closing Summary

To address shadow AI, an integrated approach is necessary. This involves designing real-time detection systems that can identify and mitigate PII exposure, along with robust policy enforcement and user education. The architecture must be scalable to handle the volume of data processed by public chatbots while maintaining high levels of security and compliance.

#### Common Mistakes
and Follow-ups
Ignoring User Education: Simply implementing technical solutions without educating users can lead to continued misuse.
Over-reliance on Policy Enforcement: Policies alone are insufficient; technical guardrails must be in place to automatically prevent data leaks.
Lack of Scalability: The architecture must handle the volume and complexity of interactions, especially during peak usage times.

**Comparison:**

```
Component	Description
Data Collection	Logs all interactions in chatbots
Detection	Uses regex and ML models to identify PII
Mitigation	Redacts or blocks sensitive information
User Education	Provides training and resources
```

**Code:**

```python
# Example Python code for PII detection using machine learning
import re

def detect_pii(text):
    patterns = {
        "name": r"[\w\s]+",  # Simple pattern to match names
        "email": r"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b",
        "address": r"(?:\d{1,4}[, ]? )*(\w+[, ]+\w*)[, ]+\d{5}"  # Basic address pattern
    }

    for key, pattern in patterns.items():
        if re.search(pattern, text):
            return {key: True}
    return {}
```

**Example input / output:**

```text
Input: User: John Doe, Chatbot Query: 'Can you provide my address at 123 Elm St?'
Output: Detected PII: {'address': True}

Input: User: Jane Smith, Chatbot Query: 'My email is jane.smith@example.com'
Output: Detected PII: {'email': True}
```

**Data:**

```json
{
  "log": [
    {
      "timestamp": "2023-10-01T14:30:00Z",
      "message": "User: John Doe, Chatbot Query: 'Can you provide my address at 123 Elm St?'"
    },
    {
      "timestamp": "2023-10-02T15:00:00Z",
      "message": "User: Jane Smith, Chatbot Query: 'My email is jane.smith@example.com'"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Collection
4. Is PII Detected?
5. Mitigation(Redaction/Blocking)
6. Log Interaction
7. Report Incident if Required
8. User Education and Training

