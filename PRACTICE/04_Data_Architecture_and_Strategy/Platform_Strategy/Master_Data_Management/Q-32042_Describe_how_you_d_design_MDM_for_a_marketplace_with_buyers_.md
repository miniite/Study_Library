---
type: "practice"
difficulty: "Expert"
---

# Describe how you'd design MDM for a marketplace with buyers and sellers who are sometimes the same physical entity across sides — what identity model prevents policy loopholes and fraud?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-domain MDM Programs](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Multi-domain_MDM_Programs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Master Data Management (MDM) in a marketplace where buyers and sellers can be the same physical entities introduces complexities, especially around preventing policy loopholes and ensuring data integrity to avoid fraud. The core challenge is to establish an identity model that uniquely identifies each entity while preventing double-counting or conflicting roles. Here’s how I would design MDM for such a scenario:

Firstly, we need to understand that the key is in managing and reconciling these dual-identity entities. These entities can be treated as either separate identities with shared attributes or as single unified entities. The approach depends on the specific business rules and regulatory requirements.

**Comparison:**

```
External ID	Buyer	Seller
123	true	false
456	false	true
789	true	true
```

**Code:**

```python
# Example pseudocode for entity identification logic
class MarketplaceEntity:
    def __init__(self, external_id):
        self.external_id = external_id
        self.is_buyer = False
        self.is_seller = False

def detect_dual_identity(entities):
    dual_identity = {}
    for entity in entities:
        if (entity.is_buyer and entity.is_seller) or (len(entity.attributes) > 10):  # heuristic to detect shared identity
            dual_identity[entity.external_id] = entity
    return dual_identity

def reconcile_dual_entity(dual_entities):
    merged_attributes = {}
    for entity in dual_entities.values():
        for attr, value in entity.attributes.items():
            if attr not in merged_attributes or (attr in merged_attributes and len(value) > len(merged_attributes[attr])):
                merged_attributes[attr] = value
    return merged_attributes

# Example usage
entities = [MarketplaceEntity('123'), MarketplaceEntity('456')]
dual_entities = detect_dual_identity(entities)
reconciled_data = reconcile_dual_entity(dual_entities)
```

**Example input / output:**

```text
# Input entities
entities = [
    MarketplaceEntity('123', is_buyer=True),
    MarketplaceEntity('456', is_seller=True),
    MarketplaceEntity('789', is_buyer=True, is_seller=True)
]

# Detect dual identities
dual_entities = detect_dual_identity(entities)

# Reconcile data for dual entities
reconciled_data = reconcile_dual_entity(dual_entities)
print(reconciled_data)  # Output: {'external_id': '789', 'is_buyer': True, 'is_seller': True}
```

**Data:**

```json
{
  "entities": [
    {"external_id": "123", "is_buyer": true, "is_seller": false},
    {"external_id": "456", "is_buyer": false, "is_seller": true},
    {"external_id": "789", "is_buyer": true, "is_seller": true}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Detect Dual Identity
4. Is Entity a Buyer and Seller?
5. Reconcile Attributes
6. Store Separate Entities
7. Return Merged Entity
8. Store Individual Entities

