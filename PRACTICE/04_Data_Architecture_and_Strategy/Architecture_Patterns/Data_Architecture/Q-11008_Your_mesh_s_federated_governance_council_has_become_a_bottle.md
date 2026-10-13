---
type: "practice"
difficulty: "Deep"
---

# Your mesh's federated governance council has become a bottleneck approving every new PII classification, so teams route around it. As principal, how do you architect policy-as-code with automated PII detection and default-deny so governance is enforced at write time without a human in the loop for the common case?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Proving PII Residency](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Proving_PII_Residency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the bottleneck issue in our mesh's federated governance council, we need to architect a system that automatically enforces policy-based control over PII data at write time. This will shift the governance enforcement from manual approval to automated processes, ensuring compliance while minimizing delays and reducing friction for teams.

Triaging the Problem

The key challenge here is to design an architecture where:

Automated PII Detection: We need a robust mechanism to identify PII in data before it's written.
Default-Deny Policy Enforcement: By default, all new classifications should be treated as sensitive until explicitly allowed by policy.
Governance Integration: Ensure that the system can integrate with existing governance frameworks and workflows seamlessly.

#### Assumptions
We are working within a cloud-native environment (e.g., AWS, Azure).
Existing data meshes leverage microservices architecture for data ingestion, transformation, and storage.
Teams have already established a set of PII classifications and policies to govern them.

#### First-Principles Solution
Step 1: Define the Policy-as-Code Framework

We will use Kubernetes Operator pattern or serverless functions (e.g., AWS Lambda) to manage policy enforcement. This approach ensures that governance rules are codified in a declarative manner, making it easier to automate and scale.

YAML
# Example of a simple policy definition in YAML
apiVersion: security.mesh.com/v1
kind: PIIProtectionPolicy
metadata:
  name: default-pii-policy
spec:
  enforcementMode: default-deny
  rules:
    - classification: "personal_id"
      allowedWorkloads: ["finance-app", "hr-system"]
Step 2: Implement Automated PII Detection

We can leverage machine learning models or pre-built libraries for PII detection. For simplicity, let's use a combination of regex and ML models.

PYTHON
# Example Python code for PII detection using regular expressions and ML model
import re
from sklearn.feature_extraction.text import CountVectorizer

def detect_pii(text):
    # Using regex patterns to identify common PII types
    pii_patterns = [r'\b\d{3}-\d{2}-\d{4}\b', r'[A-Z]{2,8}[0-9]{6}', r'\b[A-Z][a-z]+\b']  # Social Security Number, Driver's License, Names

    for pattern in pii_patterns:
        matches = re.findall(pattern, text)
        if matches:
            return True

    # Using ML model to further classify PII
    vectorizer = CountVectorizer()
    features = vectorizer.fit_transform([text])
    predictions = model.predict(features)
    return "personal_id" in predictions

# Sample input and output
input_text = "John Doe has a SSN: 123-45-6789"
print(detect_pii(input_text))  # Output: True
Step 3: Integrate with Data Mesh Write Operations

We need to integrate the PII detection logic into the data ingestion pipeline. This can be achieved by modifying the microservices responsible for writing data.

PYTHON
# Example of integrating PII detection in a write operation
def write_data_to_mesh(data, metadata):
    if detect_pii(data['value']):
        # Check against policy rules
        allowed = check_policy(metadata)
        if not allowed:
            raise ValueError("Data violates default-deny policy")

    # Proceed with writing data to the mesh
    # ...
Step 4: Automate Governance Enforcement

Using Kubernetes Operators, we can automate the enforcement of policies. For example, a Custom Resource Definition (CRD) can be created for PIIPolicy which triggers an operator to enforce rules.

YAML
# CRD definition in YAML
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: piipolicys.security.mesh.com
spec:
  group: security.mesh.com
  versions:
    - name: v1
      served: true
      storage: true
  names:
    plural: piipolicys
    singular: piipolicy
    kind: PIIProtectionPolicy

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing a policy-as-code framework, we ensure that governance is enforced automatically at write time. This reduces the bottleneck in our federated governance council and ensures compliance without human intervention for common cases.

#### Common Mistakes

Not properly defining the scope of PII detection.
Overlooking edge cases where data might be misclassified.
Failing to handle complex policy scenarios that require manual review.

#### Follow-Ups

How do you ensure accuracy in automated PII detection?
What happens if a false positive occurs during write operations?

**Comparison:**

| Classification  | Allowed Workloads                     |
|-----------------|---------------------------------------|
| personal_id     | finance-app, hr-system                 |
| email_address   | marketing-department                   |

**Code:**

```python
# Python code for PII detection and policy enforcement
import re

def detect_pii(text):
    pii_patterns = [r'\b\d{3}-\d{2}-\d{4}\b', r'[A-Z]{2,8}[0-9]{6}', r'\b[A-Z][a-z]+\b']  # Social Security Number, Driver's License, Names
    for pattern in pii_patterns:
        matches = re.findall(pattern, text)
        if matches:
            return True

    vectorizer = CountVectorizer()
    features = vectorizer.fit_transform([text])
    predictions = model.predict(features)
    return "personal_id" in predictions

def check_policy(metadata):
    # Check policy rules against metadata
    allowed_workloads = [w.name for w in metadata.allowedWorkloads]
    if metadata.classification not in allowed_workloads:
        return False
    return True
```

**Example input / output:**

```text
Input Data: {"key": "personal_id", "value": "123-45-6789"}
Output: True (PII detected and default-deny policy enforced)
```

**Data:**

```yaml
# Example CRD definition
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: piipolicys.security.mesh.com
spec:
  group: security.mesh.com
  versions:
    - name: v1
      served: true
      storage: true
  names:
    plural: piipolicys
    singular: piipolicy
    kind: PIIProtectionPolicy
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Data Ingestion
6. Is PII Detected?
7. Policy Check
8. Policy Allowed?
9. Reject Write
10. Write to Mesh

