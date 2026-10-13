---
type: "practice"
difficulty: "Expert"
---

# How do you architect encryption and key management across a lake so that per-tenant/per-region key isolation and crypto-shredding are possible without wrecking query performance or operational complexity?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant Data Platform Isolation](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Multi-tenant_Data_Platform_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're tasked with designing an encryption and key management strategy for a data lake that supports multiple tenants across different regions while ensuring per-tenant/per-region key isolation and enabling crypto-shredding, all without significantly impacting query performance or operational complexity. The solution involves leveraging multi-layered encryption techniques, robust key management practices, and a well-thought-out architecture.

#### Triage

The core question revolves around maintaining data security through encryption while balancing the trade-offs between operational complexity and query performance in a distributed environment. We need to ensure that keys are isolated per tenant and region, which means each tenant should have its own set of keys, and keys should be managed independently within regions. Additionally, we must support crypto-shredding (permanently deleting data by removing the keys) without affecting ongoing queries.

#### First-Principles Solution
Key Management System (KMS): Implement a KMS to manage encryption keys across different tenants and regions.
Tenant-Level Key Isolation: Ensure each tenant has its own set of keys, managed independently.
Region-Specific Key Stores: Store these keys in region-specific secure vaults to ensure isolation.
Query Optimization: Use techniques like caching, indexing, and query optimization to maintain performance while encrypting data.
Step-by-Step Solution
1. Key Management System (KMS)

Choose a robust KMS that supports multiple tenants and regions, such as AWS KMS or Azure Key Vault. These services offer features for key lifecycle management, including generating, storing, using, and revoking keys.

Key Management Service
Tenant ID
Tenant-Specific Keys
Region-Specific Vault
2. Tenant-Level Key Isolation

Each tenant should have its own set of keys, which are generated and managed independently within the KMS.

Key Management Service
Tenant ID
Tenant-Specific Keys
Region-Specific Vault
Per-Region Key Stores
3. Region-Specific Key Stores

Keys should be stored in region-specific secure vaults to ensure isolation and compliance with regional data regulations.

Key Management Service
Tenant ID
Tenant-Specific Keys
Region-Specific Vault
Per-Region Key Stores
4. Query Optimization

To maintain performance, we need to optimize queries by using techniques such as caching and indexing.

Key Management Service
Tenant ID
Tenant-Specific Keys
Region-Specific Vault
Per-Region Key Stores
Caching & Indexing

#### Closing Summary

In summary, we have designed a robust data architecture that leverages a KMS for key management, ensures tenant and region-specific key isolation, and employs query optimization techniques to maintain performance. This approach balances the need for strong security with operational simplicity.

#### Common Mistakes/Follow-Ups
Failing to Isolate Keys: Not ensuring that each tenant has its own set of keys can lead to data breaches if one tenant's keys are compromised.
Ignoring Query Performance: Neglecting query optimization can result in degraded performance, especially with large datasets.
Overcomplicating Key Management: Implementing overly complex key management strategies without proper justification can increase operational overhead and reduce security.

**Comparison:**

```
Tenant	Region	Key ID
T1234567890	us-west-2	AKIAIOSFODNN7EXAMPLE
```

**Code:**

```python
# Example of generating keys using AWS KMS
import boto3

def generate_key(tenant_id):
    client = boto3.client('kms')
    response = client.create_key(
        Description=f'Key for tenant {tenant_id}',
        KeyUsage='ENCRYPT_DECRYPT'
    )
    return response['KeyId']
```

**Example input / output:**

```text
Input:
- Tenant ID: T1234567890
- Region: us-west-2

Output:
- Key ID for tenant T1234567890 in region us-west-2: AKIAIOSFODNN7EXAMPLE
```

**Data:**

```json
{
  "tenant": {
    "id": "T1234567890",
    "keyId": "AKIAIOSFODNN7EXAMPLE"
  },
  "region": {
    "name": "us-west-2",
    "vaultName": "Tenant-T1234567890-Vault"
  }
}
```

**Flow:**

1. Yes
2. No
3. Query Request
4. Is Tenant Specific?
5. Tenant Key Lookup
6. Decrypt Data
7. Global Key Lookup
8. Decrypt Data

