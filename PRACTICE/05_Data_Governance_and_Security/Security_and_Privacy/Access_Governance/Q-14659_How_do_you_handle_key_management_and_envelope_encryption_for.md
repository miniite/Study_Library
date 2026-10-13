---
type: "practice"
difficulty: "Expert"
---

# How do you handle key management and envelope encryption for a lake spanning three clouds so that a single cloud KMS outage or region loss doesn't lock all analytics teams out of their data?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-border Access Governance](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Cross-border_Access_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To ensure continuous data access even in the event of a key management system (KMS) outage or regional failure, we must implement a multi-cloud architecture with redundant key storage and failover mechanisms. This involves using envelope encryption to securely store and manage keys across multiple cloud providers, ensuring that no single point of failure can lock down all analytics teams.

#### Triaging the Question

The core challenge is to guarantee uninterrupted data access by implementing robust key management strategies in a multi-cloud environment. The interviewer is probing your understanding of:

Multi-cloud Architecture: How you design and manage infrastructure across multiple cloud providers.
Key Management Systems (KMS): Knowledge on how KMSs work and their role in securing encryption keys.
Envelope Encryption: Understanding the practice of encrypting data with a key that itself is encrypted, providing an additional layer of security.
Redundancy and Failover Mechanisms: Strategies to ensure high availability and minimize downtime.

#### Assumptions
The lake spans three cloud providers: AWS, Azure, and GCP.
There are multiple analytics teams using the data.
A KMS outage or regional failure in one cloud provider should not prevent access to data by other teams.
Data governance policies must be adhered to.

#### Solution
Step 1: Define the Encryption Strategy

We will use envelope encryption where each piece of data is encrypted with a key that is itself encrypted under multiple KMSs in different clouds. This ensures that if one cloud's KMS fails, another can take over without compromising access.

Step 2: Key Management Architecture

For each piece of data:

The outer key will be managed by a centralized KMS.
The inner key will be managed by distributed KMSs across the three clouds (AWS, Azure, GCP).

Here’s a high-level flow:

Data Encryption: When data is ingested into the lake, it is first encrypted with an inner key specific to each cloud provider's region.
Key Wrapping: The outer key, used to encrypt the inner keys, is managed by a centralized KMS in one of the clouds (e.g., AWS).
Failover Mechanism: If the centralized KMS fails, the inner keys stored in other clouds' KMSs can be used to decrypt the data.
Step 3: Redundancy and Failover
Centralized KMS as Backup: Ensure that each region has a copy of the outer key encrypted with different KMSs.
Cloud-Aware Policies: Implement policies that allow seamless switching between cloud providers in case of an outage. For example, if AWS KMS fails, the system should automatically switch to Azure or GCP’s KMS.
Step 4: Implementation Details

Key Wrapping and Storage:

JSON
{
  "data": "<encrypted_data>",
  "outer_key": "aws_kms_01",
  "inner_keys": [
    {"cloud": "aws", "region": "us-east-1", "key_id": "aws_kms_us-east-1_02"},
    {"cloud": "azure", "region": "west-us", "key_id": "azure_kms_west-us_03"},
    {"cloud": "gcp", "region": "us-central1", "key_id": "gcp_kms_us-central1_04"}
  ]
}

Decryption Flow:

On decryption, the system first checks if its KMS is available.
If not, it tries the next cloud's KMS in sequence.
Step 5: Monitoring and Alerts

Implement a monitoring system to alert on any KMS failures or regional outages. This ensures immediate actions can be taken to maintain service continuity.

#### Closing Summary

By using envelope encryption with distributed key management across multiple clouds, we can ensure that no single point of failure will lock down the analytics teams' access to data. Redundancy and failover mechanisms are crucial in maintaining high availability and minimizing downtime. This approach requires careful planning and implementation but provides a robust solution.

#### Common Mistakes/Follow-ups
Over-reliance on Centralized KMS: Ensure that there is no single point of failure by distributing keys across multiple clouds.
Failure to Implement Monitoring: Lack of real-time monitoring can lead to undetected outages, compromising data access.
Insufficient Testing: Thorough testing during development and regular drills are necessary to ensure the failover mechanisms work as expected.

**Comparison:**

| Cloud | Region | Key ID          |
|-------|--------|----------------|
| AWS   | us-east-1 | aws_kms_us-east-1_02 |
| Azure | west-us  | azure_kms_west-us_03  |
| GCP   | us-central1 | gcp_kms_us-central1_04 |

**Code:**

```python
# Example Code for Key Wrapping and Decryption

from cryptography.hazmat.primitives import serialization, hashes
from cryptography.hazmat.primitives.asymmetric import padding
from cryptography.hazmat.backends import default_backend

def wrap_key(outer_key, inner_key):
    # Wrap the inner key using the outer KMS's public key
    wrapped_key = outer_key.encrypt(
        inner_key,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None
        )
    )
    return wrapped_key

def unwrap_key(outer_key, wrapped_key):
    # Unwrap the inner key using the outer KMS's private key
    original_key = outer_key.decrypt(
        wrapped_key,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None
        )
    )
    return original_key
```

**Example input / output:**

```text
Input:
{
  "data": "encrypted_data_here",
  "outer_key": "aws_kms_01",
  "inner_keys": [
    {"cloud": "aws", "region": "us-east-1", "key_id": "aws_kms_us-east-1_02"},
    {"cloud": "azure", "region": "west-us", "key_id": "azure_kms_west-us_03"},
    {"cloud": "gcp", "region": "us-central1", "key_id": "gcp_kms_us-central1_04"}
  ]
}

Output:
Decrypted Data: decrypted_data_here
```

**Data:**

```json
{
  "data": "encrypted_data_here",
  "outer_key": "aws_kms_01",
  "inner_keys": [
    {"cloud": "aws", "region": "us-east-1", "key_id": "aws_kms_us-east-1_02"},
    {"cloud": "azure", "region": "west-us", "key_id": "azure_kms_west-us_03"},
    {"cloud": "gcp", "region": "us-central1", "key_id": "gcp_kms_us-central1_04"}
  ]
}
```

**Flow:**

1. Data Ingestion
2. Encrypt with Inner Key
3. Store Inner Keys in CloudKMSs
4. Wrap Inner Keys with OuterKey
5. Centralized KMS Management
6. Cloud Outage or RegionalFailure
7. Fallback to Next Available KMS
8. Decrypt Data Using AvailableKey

