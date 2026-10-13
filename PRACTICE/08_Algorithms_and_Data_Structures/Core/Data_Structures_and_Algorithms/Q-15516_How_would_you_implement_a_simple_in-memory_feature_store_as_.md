---
type: "practice"
difficulty: "Basic"
---

# How would you implement a simple in-memory feature store as a dictionary keyed by entity ID, and what are its lookup guarantees?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Arrays and Embedding Lookup](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Arrays_and_Embedding_Lookup.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an in-memory feature store using a dictionary (also known as a hash map or associative array), each key corresponds to an entity ID, while the value stores features associated with that entity. This structure ensures fast lookups and efficient data access.

The primary goal of implementing such a feature store is to enable quick retrieval of features for machine learning models. Since we're working at a basic level, let's assume we are using Python for our implementation. The key lookup guarantees include constant time complexity ( O(1) ), assuming no hash collisions or other issues that could degrade performance.

#### Triaging the Question

The question is essentially asking how to design and implement an in-memory data structure to store features indexed by entity IDs, with a focus on the dictionary (dictionary being our primary choice due to its efficient lookup capabilities). The interviewer might be probing your understanding of basic data structures, hash functions, and memory management.

#### First-Principles Solution
Choose a Suitable Data Structure: A Python dict is ideal for this task.
Define Entity IDs: These will serve as keys in our dictionary.
Store Features as Values: The features associated with each entity ID can be stored as values within the dictionary.
Detailed Implementation
PYTHON
# Step 1: Define a simple in-memory feature store using Python's built-in dict

class InMemoryFeatureStore:
    def __init__(self):
        # Initialize an empty dictionary to hold feature data
        self.features = {}

    def add_feature(self, entity_id, features):
        """
        Add or update the features for a given entity ID.

        :param entity_id: The unique identifier of the entity.
        :param features: A dictionary containing key-value pairs representing features.
        """
        if entity_id not in self.features:
            self.features[entity_id] = {}
        self.features[entity_id].update(features)

    def get_features(self, entity_id):
        """
        Retrieve all features for a given entity ID.

        :param entity_id: The unique identifier of the entity.
        :return: A dictionary containing all feature-value pairs associated with the entity.
        """
        return self.features.get(entity_id, {})

# Example usage:
feature_store = InMemoryFeatureStore()
feature_store.add_feature("user_1", {"age": 25, "gender": "male"})
feature_store.add_feature("user_2", {"age": 30, "gender": "female"})

print(feature_store.get_features("user_1"))
# Output: {'age': 25, 'gender': 'male'}

#### Closing Summary

In this implementation, we've created a simple in-memory feature store using Python's dict as the underlying data structure. The add_feature method allows us to add or update features for entities by their unique ID, while the get_features method retrieves all associated features efficiently.

#### Common Mistakes
and Follow-Ups
Hash Collisions: While Python dictionaries typically handle collisions well, it's important to understand how they are resolved (e.g., chaining or open addressing).
Memory Management: In a production setting, consider the size of your in-memory store and potential overflow scenarios.
Concurrency Issues: If multiple processes access this feature store simultaneously, proper synchronization mechanisms need to be implemented.

**Code:**

```python
class InMemoryFeatureStore:
    def __init__(self):
        self.features = {}

    def add_feature(self, entity_id, features):
        if entity_id not in self.features:
            self.features[entity_id] = {}
        self.features[entity_id].update(features)

    def get_features(self, entity_id):
        return self.features.get(entity_id, {})
```

**Example input / output:**

```text
feature_store = InMemoryFeatureStore()
feature_store.add_feature("user_1", {"age": 25, "gender": "male"})
feature_store.add_feature("user_2", {"age": 30, "gender": "female"})

print(feature_store.get_features("user_1"))
# Output: {'age': 25, 'gender': 'male'}
```

