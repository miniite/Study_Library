---
type: "practice"
difficulty: "Deep"
---

# How would you implement a fixed-capacity LFU cache, and what breaks LFU that LRU handles gracefully?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Eviction Policies and TTL](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Eviction_Policies_and_TTL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A fixed-capacity Least Frequently Used (LFU) cache stores items in such a way that the least frequently used item is removed when the cache exceeds its capacity. In contrast to Least Recently Used (LRU), LFU focuses on frequency rather than recency, making it more suitable for scenarios where older but still relevant data might be accessed less often.

#### Triage
This question asks you to implement an LFU cache and explain why LRU handles certain situations better. The interviewer is probing your understanding of caching strategies, data structures, and trade-offs between different algorithms. Key assumptions include the fixed capacity of the cache and the need to efficiently track usage frequency.

#### First-principles solution

Data Structures: Use a combination of a hash map for quick access to items by key and an ordered list or map for tracking frequencies.
Tracking Frequencies: Maintain a separate structure (like a list or another map) that maps each unique frequency to nodes representing the values with that frequency.
Insertion: Update the frequency counters, add new entries if they don’t exist, and move items between frequency lists as needed.
Eviction: Remove the least frequently used item(s) when the cache is full.

Let's break this down step by step:

Step 1: Define the Data Structures
CacheItem: A node representing an item in the cache, holding key, value, and frequency.
FrequencyMap: Maps frequencies to lists of CacheItems with that frequency.
Capacity (fixed capacity).
Step 2: Implement Insertion

When inserting a new item:

Check if it already exists and update its frequency.
Add the item to the FrequencyMap.
If the cache is full, evict the least frequently used item(s) from the highest frequencies first.
Step 3: Eviction Strategy

To handle eviction:

Identify all nodes with the lowest frequency in the FrequencyMap.
Remove one of these nodes (or multiple if there are ties).
Step 4: Example Implementation

Here’s a simplified example:

CPP
class LFUCache {
private:
    int capacity;
    unordered_map<int, list<CacheItem>::iterator> keyToIt; // Key to iterator in frequency map
    unordered_map<int, list<pair<int, int>>> freqMap;     // Frequency -> [(key, value)]
    int minFreq;

public:
    LFUCache(int capacity) : capacity(capacity), minFreq(0) {}

    int get(int key) {
        if (keyToIt.find(key) == keyToIt.end()) return -1;
        update(key);
        return keyToIt[key]->second.second;
    }

    void put(int key, int value) {
        if (capacity <= 0) return;

        if (keyToIt.find(key) != keyToIt.end()) {
            update(key, value);
        } else {
            if (keyToIt.size() == capacity) {
                evict();
            }
            insert(key, value);
        }
    }

private:
    void update(int key, int newValue = -1) {
        auto it = keyToIt[key];
        int oldFreq = it->second.first;
        freqMap[oldFreq].erase(it); // Remove from current frequency list

        if (freqMap[oldFreq].empty() && oldFreq == minFreq) {
            ++minFreq; // Update minFreq
        }

        CacheItem item{key, newValue, oldFreq + 1};
        keyToIt[key] = freqMap[item.freq].insert(freqMap[item.freq].begin(), make_pair(key, item));
    }

    void insert(int key, int value) {
        CacheItem item{key, value, 1};
        minFreq = 1;
        keyToIt[key] = freqMap[1].insert(freqMap[1].begin(), make_pair(key, item));
    }

    void evict() {
        auto it = freqMap[minFreq].back();
        keyToIt.erase(it.first);
        freqMap[minFreq].pop_back();
    }
};

#### Closing Summary

This implementation ensures that the LFU cache operates efficiently by maintaining an ordered list of frequencies and quickly accessing items through a hash map. The update function handles adding or updating item frequencies, while evict ensures that the least frequently used items are removed when necessary.

#### Common Mistakes

Failing to correctly update frequency counters.
Not handling edge cases like eviction from multiple lists with identical frequencies.
Poor performance due to inefficient data structure choices (e.g., frequent lookups in large unordered maps).

**Code:**

```
CPP
Copy
class LFUCache {
private:
    int capacity;
    unordered_map<int, list<CacheItem>::iterator> keyToIt; // Key to iterator in frequency map
    unordered_map<int, list<pair<int, int>>> freqMap;     // Frequency -> [(key, value)]
    int minFreq;

public:
    LFUCache(int capacity) : capacity(capacity), minFreq(0) {}

    int get(int key) {
        if (keyToIt.find(key) == keyToIt.end()) return -1;
        update(key);
        return keyToIt[key]->second.second;
    }

    void put(int key, int value) {
        if (capacity <= 0) return;

        if (keyToIt.find(key) != keyToIt.end()) {
            update(key, value);
        } else {
            if (keyToIt.size() == capacity) {
                evict();
            }
            insert(key, value);
        }
    }

private:
    void update(int key, int newValue = -1) {
        auto it = keyToIt[key];
        int oldFreq = it->second.first;
        freqMap[oldFreq].erase(it); // Remove from current frequency list

        if (freqMap[oldFreq].empty() && oldFreq == minFreq) {
            ++minFreq; // Update minFreq
        }

        CacheItem item{key, newValue, oldFreq + 1};
        keyToIt[key] = freqMap[item.freq].insert(freqMap[item.freq].begin(), make_pair(key, item));
    }

    void insert(int key, int value) {
        CacheItem item{key, value, 1};
        minFreq = 1;
        keyToIt[key] = freqMap[1].insert(freqMap[1].begin(), make_pair(key, item));
    }

    void evict() {
        auto it = freqMap[minFreq].back();
        keyToIt.erase(it.first);
        freqMap[minFreq].pop_back();
    }
};
```

