---
type: "practice"
difficulty: "Deep"
---

# An interviewer follow-up: you added a PII filter before the LLM — how do you handle its false positives destroying legitimate context and its false negatives leaking data?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ PII Filters for LLMs](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/PII_Filters_for_LLMs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In handling PII (Personally Identifiable Information) in AI models, particularly with Large Language Models (LLMs), we need to ensure that our filtering mechanisms are both precise and robust. By carefully designing a multi-layered approach involving advanced machine learning techniques and contextual understanding, we can mitigate the risks of false positives and false negatives effectively.

#### Triaging the Question

The core issue here is ensuring data privacy while maintaining the integrity and context of legitimate data during the LLM processing pipeline. The question delves into how to balance the trade-offs between precision (false positives) and recall (false negatives). This requires a deep understanding of both the technical implementation and the ethical implications.

#### First-Principles Solution
Preprocessing Layer: Implement an initial PII filter that uses advanced NLP techniques like rule-based matching, regular expressions, and context-aware models to flag potential PII.
Postprocessing Layer: Use a feedback mechanism where the LLM's responses are checked against known PII patterns or contextual clues.
User Feedback Loop: Allow users to report incorrect filtering decisions, which can then be used to fine-tune the system.
Anomaly Detection: Implement real-time anomaly detection using unsupervised learning techniques to flag suspicious patterns that might indicate false positives.
Detailed Steps

Preprocessing Layer

Rule-Based Matching: Use predefined rules and regular expressions to detect common PII formats (e.g., names, addresses, SSNs).
Context-Aware Models: Train NLP models on annotated datasets where the context is known, allowing them to understand when certain information might be PII in a specific context.

Postprocessing Layer

Response Analysis: Analyze the LLM's responses for any potential PII that was not caught by initial filters.
Contextual Checks: Use additional contextual information (e.g., previous messages, user roles) to cross-reference and verify identified PII.

User Feedback Loop

Reporting Mechanism: Provide a simple interface where users can flag false positives or negatives.
Continuous Improvement: Regularly update the filtering models based on user feedback and new data.

Anomaly Detection

Unsupervised Learning Models: Implement models like Autoencoders or Generative Adversarial Networks (GANs) to detect unusual patterns that might indicate false positives.
Real-Time Alerts: Set up alerts for suspicious activities that require immediate attention from a human analyst.

#### Closing Summary
and Common Mistakes

The key is to have a robust, multi-layered system that can handle the complexity of real-world data. False positives can destroy context, while false negatives risk leaking sensitive information. By combining rule-based systems with machine learning and user feedback loops, we can achieve a balance that minimizes both risks.

#### Common Mistakes

Over-reliance on Rules: Relying too heavily on simple rules without considering the dynamic nature of PII.
Lack of Contextual Understanding: Ignoring contextual information that could help in identifying and handling PII more accurately.
Ignoring User Feedback: Not incorporating user feedback to continuously improve the filtering mechanism.
Common Follow-Ups
How do you ensure the privacy of users' data during this process?
What measures do you take to prevent misuse of the system by malicious actors?
Can you walk me through a scenario where false positives or negatives occurred and how it was resolved?

**Comparison:**

```
PII Type	Rule-Based Matching Patterns	Context-Aware Model Use
Name	\b[A-Z][a-z]+\b	N/A
SSN	(\d{3}-\d{2}-\d{4})	Yes (trained on context)
Address	[0-9]+ [A-Za-z\s]+[0-9,]*	Yes (contextual analysis)
Email	\b[A-Z0-9._%+-]+@[A-Z0-9.-]+.[A-Z]{2,}\b	N/A
```

**Code:**

```python
# Example PII Detection Function using Rule-Based Matching

def detect_pii(text):
    rules = {
        'name': r'\b[A-Z][a-z]+\b',  # Simple name rule
        'ssn': r'(\d{3}-\d{2}-\d{4})',  # Social Security Number pattern
    }

    for key, pattern in rules.items():
        matches = re.findall(pattern, text)
        if matches:
            return {key: matches}
    return {}
```

**Example input / output:**

```text
Input:
"Hello, my name is John Doe and my SSN is 123-45-6789. How can I help you?"

Output:
{
  "piis_detected": [
    {
      "type": "name",
      "value": ["John Doe"]
    },
    {
      "type": "ssn",
      "value": ["123-45-6789"]
    }
  ]
}
```

**Data:**

```json
{
  "user_input": "Hello, my name is John Doe and my SSN is 123-45-6789. How can I help you?",
  "piis_detected": [
    {
      "type": "name",
      "value": ["John Doe"]
    },
    {
      "type": "ssn",
      "value": ["123-45-6789"]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Input
4. Preprocessing Layer
5. Is PII Detected?
6. Postprocessing Layer
7. No Action
8. User Feedback Loop
9. Improve Models
10. Anomaly Detection
11. Real-Time Alerts

