---
type: "practice"
difficulty: "Expert"
---

# Your feature store has a real-time and offline half. How do you guarantee consistent PII masking/consent enforcement across both without training-serving skew?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Consent Enforcement](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/Consent_Enforcement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure consistent PII masking and consent enforcement across both real-time and offline halves of a feature store, we must architect a system that enforces data privacy at every stage while maintaining consistency between training and serving pipelines. This involves several key steps: designing an integrated data governance framework, implementing a unified masking policy, and ensuring compliance with real-time updates.

The core challenge here is to prevent any discrepancies or delays in PII handling as data flows from the source to the feature store and then into both training and serving environments. Let’s break down the solution step by step:

Unified Data Governance Framework: Establish a centralized governance framework that oversees all aspects of data privacy, including data masking, consent management, and audit trails.

#### Cons
istent Masking Policy: Define and enforce consistent PII masking rules across both real-time and offline systems.
Real-Time Compliance Enforcement: Ensure real-time updates are masked and consented to before they reach the feature store.
Data Consistency Checks: Implement checks to ensure consistency between training and serving data.

Let’s dive into each of these steps in detail:

1. Unified Data Governance Framework

A robust governance framework ensures that all stakeholders, including developers, data engineers, and business teams, adhere to strict privacy policies. This includes:

Data Masking Policies: Define how PII should be masked based on the sensitivity level.

#### Cons
ent Management: Track user consent for using their personal information in training datasets.
Audit Trails: Maintain logs of all actions taken by users, including masking and consent updates.
2. Consistent Masking Policy

To ensure consistency, we need to implement a policy that is applied uniformly across both real-time and offline systems:

Rule-Based Engine: Develop or use an existing rule engine to apply consistent PII masking rules based on predefined policies.
Dynamic Consent Handling: Implement mechanisms to dynamically handle consent for new users and changes in user preferences.
3. Real-Time Compliance Enforcement

Real-time updates must be masked and consented to before they are ingested into the feature store:

Pre-Ingestion Masking: Before data enters the feature store, apply masking rules to ensure PII is handled correctly.

#### Cons
ent Verification: Verify user consent for real-time data ingestion by checking against a central consent database.
4. Data Consistency Checks

Ensure that training and serving data are consistent:

Data Synchronization Mechanism: Implement a mechanism to synchronize the feature store with both training and serving systems, ensuring that any changes in real-time updates are reflected in both.
Periodic Audit: Regularly perform audits to check for discrepancies between training and serving datasets.

#### Triaging the Question

The interviewer is probing your understanding of:

Data Privacy Principles: Ensuring data privacy across different stages of data processing.

#### Cons
istency Across Systems: Maintaining consistency in PII handling between real-time and offline systems.
Enforcement Mechanisms: Implementing robust mechanisms to enforce data policies.
Step-by-Step Solution
1. Unified Data Governance Framework
Data Source
Real-Time Ingestion
Feature Store
Training & Serving Pipelines
Audit Logs
2. Consistent Masking Policy
PYTHON
def mask_pii(data, rules):
    # Apply masking based on predefined rules
    return [mask_data(item, rule) for item, rule in zip(data, rules)]

def verify_consent(user_id, consent_db):
    # Check user consent status from the database
    return consent_db.get_consent_status(user_id)
3. Real-Time Compliance Enforcement
PYTHON
def preprocess_real_time_data(real_time_data, masking_rules, consent_db):
    masked_data = mask_pii(real_time_data, masking_rules)
    if verify_consent(user_id=real_time_data['user_id'], consent_db=consent_db):
        # Masked and consented data can be ingested into feature store
        return masked_data
    else:
        raise Exception("Consent not granted")
4. Data Consistency Checks
PYTHON
def ensure_data_consistency(training_data, serving_data, sync_mechanism):
    # Synchronize training and serving datasets
    sync_mechanism.synchronize_datasets(training_data, serving_data)

#### Closing Summary

By establishing a unified data governance framework, implementing consistent masking policies, ensuring real-time compliance enforcement, and maintaining data consistency checks, we can guarantee that PII is handled consistently across both the real-time and offline halves of our feature store. This approach ensures no discrepancies or delays in PII handling, thereby preventing training-serving skew.

#### Common Mistakes/Follow-Ups
Overlooking Real-Time Updates: Failing to properly mask and consent real-time updates can lead to inconsistencies.
Inadequate Consent Handling: Not managing user consent dynamically can result in unauthorized data usage.
Failure to Synchronize Data: Inconsistent synchronization between training and serving datasets can introduce skew.

By addressing these common issues, we ensure a robust solution for maintaining consistent PII handling across the feature store.

**Comparison:**

| Rule Type | Apply To         |
|-----------|-----------------|
| partial_masking | [email, phone_number] |

**Code:**

```python
def mask_pii(data, rules):
    # Apply masking based on predefined rules
    return [mask_data(item, rule) for item, rule in zip(data, rules)]

def verify_consent(user_id, consent_db):
    # Check user consent status from the database
    return consent_db.get_consent_status(user_id)

def preprocess_real_time_data(real_time_data, masking_rules, consent_db):
    masked_data = mask_pii(real_time_data, masking_rules)
    if verify_consent(user_id=real_time_data['user_id'], consent_db=consent_db):
        # Masked and consented data can be ingested into feature store
        return masked_data
    else:
        raise Exception("Consent not granted")

def ensure_data_consistency(training_data, serving_data, sync_mechanism):
    # Synchronize training and serving datasets
    sync_mechanism.synchronize_datasets(training_data, serving_data)
```

**Example input / output:**

```text
Input (Real-Time Data):
{
  "user_id": 12345,
  "name": "John Doe",
  "email": "johndoe@example.com",
  "phone_number": "9876543210"
}

Output (Masked Real-Time Data):
{
  "user_id": 12345,
  "name": "John Doe",
  "email": "john.d***@example.com",
  "phone_number": "98******10"
}
```

**Data:**

```yaml
data:
  user_id: 12345
  real_time_data:
    - name: John Doe
      email: johndoe@example.com
      phone_number: 9876543210
  masking_rules:
    - rule_type: partial_masking
      apply_to: [email, phone_number]
```

**Flow:**

1. Data Source
2. Real-Time Ingestion
3. Feature Store
4. Training & Serving Pipelines
5. Audit Logs

