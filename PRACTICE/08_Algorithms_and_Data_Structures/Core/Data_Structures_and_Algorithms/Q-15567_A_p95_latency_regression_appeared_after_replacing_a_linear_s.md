---
type: "practice"
difficulty: "Deep"
---

# A p95 latency regression appeared after replacing a linear scan with a HashMap in a feature-lookup hot loop; what production factors (cache misses, resize, GC) could make the 'faster' structure slower and how do you verify?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Hash Map Performance Pitfalls](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Map_Performance_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with a performance issue where replacing a linear scan with a HashMap in a feature-lookup hot loop has led to a p95 latency regression. This is not uncommon and can be attributed to several production factors such as cache misses, resize operations, and garbage collection (GC) overheads. Understanding these factors and verifying their impact step-by-step will help us diagnose the issue accurately.

The core idea here is that while HashMaps offer average constant-time complexity for lookups, they introduce additional overheads like hash collisions, resizing during inserts/deletes, and GC pauses. Let's break down each factor to see how it could affect performance:

Cache Misses: A HashMap can lead to cache misses if the hash function does not distribute keys well across memory locations.
Resize Operations: Resizing a HashMap is an expensive operation because it involves creating a new array and copying all entries over, which can significantly increase latency during peak times.
Garbage Collection (GC): Frequent object creations (like key-value pairs in the HashMap) and potential GC pauses can degrade performance.

To verify these factors, we need to perform a thorough analysis that includes both theoretical reasoning and empirical evidence from production data.

**Code:**

```java
// Example code snippet for feature-lookup using HashMap
import java.util.HashMap;

public class FeatureLookup {
    private final HashMap<Integer, String> features = new HashMap<>();

    public void initFeatures() {
        // Populate the HashMap with features
        int size = 1000; // Initial capacity
        for (int i = 0; i < size; i++) {
            features.put(i, "feature" + i);
        }
    }

    public String getFeature(int id) {
        return features.get(id);
    }
}
```

