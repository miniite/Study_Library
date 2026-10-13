---
type: "practice"
difficulty: "Deep"
---

# Explain how open addressing with linear probing handles deletions and why tombstones are needed.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stream Join Data Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Open addressing with linear probing is a method of handling collisions in hash tables by sequentially searching through the table starting from the hashed index. However, this method poses challenges when trying to delete entries without breaking the linear probing chain. To address these issues, tombstones are introduced as a marker for deleted entries, allowing the table to maintain its efficiency and avoid false positives.

#### Triaging the Question

The question asks us to explain how open addressing with linear probing handles deletions and why tombstones (also known as deletion markers) are necessary. The core of this question revolves around understanding the limitations of linear probing when it comes to deleting entries, and how using tombstones mitigates these issues.

#### First-Principles Solution
Understanding Linear Probing

Linear probing works by placing an item in the next available slot after a collision. This means that all items are stored directly in the hash table without pointers or linked lists. However, this simplicity comes with a significant challenge when it comes to deletions: once an entry is deleted, its slot needs to be reused while still keeping the remaining entries accessible via linear probing.

The Problem of Deletions

When a deletion occurs, the slot left by the deleted item can disrupt the sequential search required for probing. For example, consider a scenario where we delete item A from index 5 in a hash table that uses linear probing. If subsequent items are inserted or accessed via linear probing, they might not find their intended positions if the slot of item A is simply left empty.

Introducing Tombstones

To solve this problem, tombstones (or deletion markers) are used. Instead of removing the item from its slot, a tombstone is placed in the deleted slot to indicate that the position should be skipped during linear probing. This helps maintain the integrity of the probing sequence and prevents false positives.

Detailed Steps for Deletion with Tombstones
Hash and Start Probing: When inserting or accessing an entry, start from the hashed index.
Handle Collisions by Probing: Sequentially check each slot until you find an empty one or a tombstone.
Deletion Process:
Step 1: Hash the key to get the initial index.
Step 2: Start probing from this index.
Step 3: If an item is found, mark it as deleted by placing a tombstone in its slot.
Accessing and Inserting:
Step 1: During insertion, if you encounter a tombstone during probing, treat it as empty and place the new entry there.
Why Tombstones are Needed

Tombstones are essential because they allow the hash table to maintain a consistent probing sequence even after deletions. Without tombstones, simply marking an item as deleted could lead to false positives or broken chains, disrupting the linear probing mechanism.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, open addressing with linear probing uses tombstones to handle deletions effectively by maintaining the integrity of the probing sequence and avoiding false positives. Key points include:

Tombstones mark deleted slots during deletion.
They ensure that all items remain accessible via sequential probing.

#### Common mistakes
often involve failing to recognize how tombstones maintain the probing chain, leading to issues with false positives or broken sequences. Follow-up questions might explore scenarios where multiple deletions occur and how tombstones affect performance in such cases.

Depth + Language Match

The explanation is detailed enough for a deep understanding of the topic, focusing on both the technical aspects and the practical implications of using tombstones in linear probing.

**Code:**

```python
def probe_table(table, index):
    i = 0
    while True:
        current_index = (index + i) % table.size
        if table[current_index] is None or table[current_index] == Tombstone():
            return current_index
        i += 1

class Tombstone:
    pass

# Example usage
table_size = 10
hash_table = [None] * table_size
tombstone = Tombstone()

# Insert items with linear probing
for key in ["A", "B", "C"]:
    index = hash(key)
    probe_index = probe_table(hash_table, index)
    if hash_table[probe_index] is None:
        hash_table[probe_index] = key

# Delete an item using tombstones
hash_table[hash("A")] = tombstone

print([item or "TOMBSTONE" for item in hash_table])
```

**Flow:**

1. Yes
2. Tombstone
3. Hash Key
4. Compute Index
5. Probe Table
6. Insert/New Entry
7. Treat as Empty Slot
8. Continue Probing
9. Find Slot for Insertion

