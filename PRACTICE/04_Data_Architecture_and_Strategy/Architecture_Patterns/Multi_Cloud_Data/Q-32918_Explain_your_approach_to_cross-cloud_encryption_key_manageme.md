---
type: "practice"
difficulty: "Expert"
---

# Explain your approach to cross-cloud encryption key management — do you centralize on one cloud's KMS, run an external KMS, or federate, and what does each choice do to your blast radius?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Hub-and-spoke Cross-cloud Networking](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Hub-and-spoke_Cross-cloud_Networking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage encryption keys across multiple clouds in a multi-cloud environment, we must consider several factors including security, compliance, cost, and operational complexity. The three primary strategies are centralizing on one cloud's Key Management Service (KMS), using an external KMS, or federating the key management processes. Each approach has its trade-offs, particularly in terms of blast radius—how far-reaching the impact is if something goes wrong.

Centralizing encryption keys on a single cloud’s KMS can simplify management and minimize complexity but introduces a potential single point of failure. Running an external KMS offers more flexibility and control over key lifecycle management but may increase operational overhead. Federating key management processes across clouds provides enhanced resilience and compliance with regional regulatory requirements, albeit at the cost of increased operational complexity.

**Comparison:**

```
Strategy	Blast Radius	Security	Compliance	Cost
Centralized KMS	Limited to the centralized cloud region	High, but risk of single point of failure	Good if all regions covered	Lower
External KMS	Controlled by external provider	Higher flexibility and control	May require additional compliance measures	Higher
Federated	Spread across multiple clouds	Resilience, regional governance	Complex compliance management	Highest
```

**Code:**

```python
# Example Python code to interact with a central KMS in AWS
import boto3

def get_key_from_aws_kms(key_id):
    client = boto3.client('kms')
    response = client.get_key_pair(
        KeyId=key_id,
        GrantTokens=['string']
    )
    return response['Plaintext']

# Example Python code to interact with an external KMS
import requests

def get_key_from_external_kms(key_id):
    url = f"https://external-kms.example.com/api/v1/key/{key_id}"
    response = requests.get(url, auth=('username', 'password'))
    return response.json()['key_material']
```

**Example input / output:**

```text
# Example input for getting a key from AWS KMS
Input (Python):
key = get_key_from_aws_kms("1234abcd-12ab-34cd-56ef-1234567890ab")

Output:
b'\x89\xac\xde\x0d\x1a'
```

**Data:**

```json
{
  "centralized_kms": {
    "cloud_provider": "AWS",
    "region": "us-west-2",
    "kms_key_id": "1234abcd-12ab-34cd-56ef-1234567890ab"
  },
  "external_kms": {
    "url": "https://external-kms.example.com/api/v1/key/{key_id}",
    "auth": {"username": "admin", "password": "secretpassword"}
  }
}
```

**Flow:**

1. Centralized KMS
2. Single Point of Failure
3. Simultaneous Access AcrossRegions
4. Lower Compliance Complexity
5. External KMS
6. Higher Flexibility and Control
7. Operational Overhead
8. Regional Compliance
9. Federated KMS
10. Enhanced Resilience
11. Increased OperationalComplexity
12. Multiregional Compliance

