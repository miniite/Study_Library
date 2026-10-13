---
type: "practice"
difficulty: "Deep"
---

# Your GenAI observability (prompt logging, trace capture, token accounting, eval scores) now stores every request-response pair and the storage plus processing cost is climbing fast. How do you govern AI-observability cost — sampling, retention, PII-safe redaction — without losing the traces you need to debug quality regressions?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Spend Freezes and Prioritization](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Spend_Freezes_and_Prioritization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively govern the costs associated with GenAI observability while maintaining necessary trace data, we need a strategic approach that balances cost management and operational needs. The key involves implementing sampling strategies, optimizing data retention policies, and ensuring sensitive information is redacted appropriately.

First, let’s define what our current situation entails: every request-response pair, including prompt logging, trace capture, token accounting, and eval scores, is being stored in detail. This comprehensive approach provides full visibility but also incurs significant storage and processing costs. Our goal is to maintain the integrity of the observability data while reducing these costs.

#### Triaging the Question

The interviewer is probing how we can optimize our cost governance strategy without compromising on the quality of debugging tools needed for maintaining GenAI systems. We need to consider:

Sampling Techniques: How do we sample the data to reduce storage and processing costs?
Data Retention Policies: What are the right retention periods for different types of data to balance between cost and utility?
PII Redaction: How can we ensure sensitive information is redacted while preserving necessary context?

#### First-Principles Solution
1. Sampling Techniques

To reduce storage costs, implementing a sampling strategy is crucial. We can use techniques like:

Random Sampling: A fixed percentage of all data points are stored.
Threshold-based Sampling: Data points above a certain threshold in terms of eval scores or trace length are stored.
Behavioral-Based Sampling: Focus on interactions that exhibit unusual behavior, which is more likely to contain bugs.

For instance, we can set up a random sampling rate of 10% for all request-response pairs. For traces with eval scores above a certain threshold (e.g., those indicating suboptimal performance), we could store 100% of these traces.

2. Data Retention Policies

Retention policies should be tailored to the type and importance of data:

Short-Term Traces: Store for a few hours or days if they are part of recent user interactions.
Long-Term Debugging Needs: Retain more extensive logs for debugging purposes, but set clear expiration dates (e.g., 30-90 days).
Historical Analysis: Use archived data for periodic analysis and training model improvements.
3. PII Redaction

For sensitive information like personally identifiable information (PII), we must implement robust redaction techniques:

Tokenization: Replace sensitive tokens with placeholders.
Hashing: Use non-reversible hashes to store sensitive information securely.
Anonymization Techniques: Ensure that re-identification is prevented while still maintaining necessary context for debugging.

#### Closing Summary
and Common Mistakes

By implementing a well-thought-out sampling strategy, optimizing data retention policies, and ensuring proper PII redaction, we can effectively govern the costs associated with GenAI observability. The key is to balance these measures to ensure that critical data remains accessible while minimizing storage and processing overhead.

#### Common mistakes
include:

Overzealous Sampling: Not retaining enough data for debugging needs.
Insufficient Redaction Policies: Failing to adequately protect sensitive information, leading to potential compliance issues.
Inadequate Retention Management: Excessively storing unnecessary data, driving up costs without providing significant benefits.
Final Self-Check

The solution is both comprehensive and balanced, addressing cost management while preserving necessary operational data. It ensures a strategic approach that aligns with both technical and business requirements.

**Comparison:**

```
Data Type	Retention Period (days)	Sampling Rate
General Interactions	7	10%
Debugging Needs	30	100%
Historical Analysis	Archive	As needed
```

**Code:**

```python
# Example of implementing random sampling in code
import random

def sample_data(data):
    sample_rate = 0.1
    sampled_data = [x for i, x in enumerate(data) if random.random() < sample_rate]
    return sampled_data

# Example of redacting PII
def redact_pii(text):
    import re
    pii_pattern = r'\b[A-Z][a-z]+\s+[A-Z][a-z]+@[A-Za-z]+\.[A-Za-z]{2,}\b'  # Simple email pattern
    anonymized_text = re.sub(pii_pattern, "REDACTED_EMAIL", text)
    return anonymized_text

# Example of retention policy management (pseudo-code)
def manage_retention(data):
    current_time = datetime.now()
    cutoff_date = current_time - timedelta(days=30)  # Retain data for 30 days
    retained_data = [x for x in data if datetime.strptime(x['timestamp'], '%Y-%m-%d %H:%M:%S') > cutoff_date]
    return retained_data
```

**Example input / output:**

```text
Input:
Sample request-response pairs with varying eval scores and timestamps.

Output:
Randomly sampled data for evalScore > 0.95, retained for debugging needs within the last 30 days.
Redacted text where PII is replaced by "REDACTED_EMAIL".
```

**Data:**

```json
{
  "sampledData": [
    {"prompt": "Example prompt", "response": "Sample response", "evalScore": 0.8, "timestamp": "2023-10-01T12:00:00Z"},
    {"prompt": "Another example", "response": "More sample data", "evalScore": 0.95, "timestamp": "2023-10-02T14:00:00Z"}
  ],
  "retainedData": [
    {"prompt": "Important prompt", "response": "Critical response", "evalScore": 0.7, "timestamp": "2023-10-01T12:00:00Z"},
    {"prompt": "Less important interaction", "response": "Standard feedback", "evalScore": 0.85, "timestamp": "2023-10-02T14:00:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is evalScore > threshold?
5. Sample 100%
6. Randomly Sample 10%
7. Redact PII
8. Manage Retention Based onTime
9. Store Data for Debugging
10. Store Sampled Data
11. Archive Historical Data
12. End
13. End
14. End

