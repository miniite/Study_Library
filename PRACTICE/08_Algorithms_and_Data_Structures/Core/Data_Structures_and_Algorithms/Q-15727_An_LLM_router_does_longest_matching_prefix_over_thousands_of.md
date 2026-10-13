---
type: "practice"
difficulty: "Deep"
---

# An LLM router does 'longest matching prefix' over thousands of prompt-routing rules to pick a model; why a radix trie over linear scan, and how do you keep updates lock-free while requests read the table?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Tries and Prefix Matching](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tries_and_Prefix_Matching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an LLM (Large Language Model) router, the task is to efficiently route incoming prompts to the appropriate model based on predefined prompt-routing rules. The challenge lies in processing thousands of such rules with minimal latency while ensuring that updates can be made without disrupting ongoing operations. A radix trie offers a compelling solution over a linear scan due to its inherent properties and performance characteristics.

First, let’s triage the question: the core issue here is understanding why a radix trie is preferred over a simple linear search for prompt routing rules, especially in terms of scalability and concurrency management.

#### Triaging the Question
Why Radix Trie?
A radix trie (or prefix tree) allows efficient longest matching prefix lookup.
It reduces the number of comparisons needed by leveraging the structure to prune branches early.
Concurrency Considerations:
Updates must be made lock-free while reads are performed concurrently.

#### First-Principles Solution
1. Why Radix Trie?

A radix trie is a tree-like data structure that stores keys by their prefixes, allowing for efficient prefix matching and searching. Here’s why it's better than linear scans:

Efficiency: For (n) rules with an average length of (k), a linear scan might require comparing each rule to the prompt until a match is found or all rules are exhausted. This can be (O(n \times k)). A radix trie, on the other hand, can find the longest matching prefix in (O(k + m)) where (m) is the length of the current query.
Scalability: As the number of rules grows, a linear scan becomes increasingly inefficient. Radix tries remain efficient because they structure data to minimize the number of comparisons needed.
2. Lock-Free Updates

To ensure lock-free updates while allowing concurrent reads, we can use a combination of techniques:

Hash-Based Indices: Maintain a hash table that maps each rule ID (or any unique identifier) to its corresponding node in the radix trie.
Concurrent Data Structures:
Use a concurrent hash map for quick access and update operations.
For nodes, consider using atomic or lock-free data structures like AtomicReference (Java), std::atomic (C++), or equivalent in other languages.

#### Example
Workflow
Insertion:
Insert new rules into the radix trie, updating the hash table for quick lookup.
Deletion:
Mark the rule node as deleted and update the hash table to reflect this state.
Lookup:
Use the hash table to quickly find the root node corresponding to a given rule ID.
Traverse the radix trie from the root, checking each branch until a match is found or all branches are exhausted.
Sample Code

Here’s an example implementation in pseudocode:

PYTHON
class RadixTrieNode:
    def __init__(self):
        self.children = {}
        self.is_end_of_rule = False

class Trie:
    def __init__(self):
        self.root = RadixTrieNode()
        self.rule_map = {}  # Hash map to quickly access nodes by rule ID

    def insert(self, rule_id, rule):
        current_node = self.root
        for char in rule:
            if char not in current_node.children:
                current_node.children[char] = RadixTrieNode()
            current_node = current_node.children[char]
        current_node.is_end_of_rule = True
        self.rule_map[rule_id] = current_node

    def delete(self, rule_id):
        node_to_delete = self.rule_map.pop(rule_id)
        # Implement logic to mark nodes as deleted if necessary

    def lookup(self, prompt):
        current_node = self.root
        for char in prompt:
            if char not in current_node.children:
                return None
            current_node = current_node.children[char]
        return current_node.is_end_of_rule

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary

A radix trie offers a robust solution for efficiently managing and routing prompts based on thousands of rules. By leveraging its structured nature, it minimizes the number of comparisons needed for lookups. Concurrent updates can be managed using hash maps and atomic data structures to ensure thread safety without blocking reads.

#### Common Mistakes/Follow-Ups
Ignoring Atomic Operations: Failing to use atomic operations or lock-free data structures can lead to race conditions.
Not Using Hash Tables: Overly complex implementations might bypass the efficiency gains of hash tables, leading to unnecessary complexity.
Incomplete Handling of Deletions: Ensuring that nodes are correctly marked and garbage collected is crucial for maintaining the trie’s integrity.

By addressing these points, you can build a highly efficient and scalable system for prompt routing in LLMs.

