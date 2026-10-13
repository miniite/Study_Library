---
type: "practice"
difficulty: "Intermediate"
---

# Why would you use a trie to implement autocomplete over a set of known prompt commands, and what is its lookup complexity per prefix?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Tries and Prefix Matching](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tries_and_Prefix_Matching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: A Trie (prefix tree) is an efficient data structure used for implementing autocomplete systems. It provides fast prefix-based lookups with a time complexity of O(L), where L is the length of the prefix.

When considering an autocomplete system for known prompt commands, using a Trie offers several advantages over other methods like hash maps or arrays. The key benefits include faster prefix matching and more efficient storage for strings that share common prefixes. This makes Trie particularly suitable when dealing with a large number of short words or phrases.

#### Triage

Question Focus: We need to understand why a Trie is preferred in implementing an autocomplete system over other data structures.
Core Concepts: Autocomplete systems, Trie structure, lookup complexity, and storage efficiency.

#### Assumptions
The prompt commands are known and can be represented as strings.

Step-by-Step Solution:

Understanding Autocomplete Systems: An autocomplete system provides suggestions based on the prefix of a user's input. For example, if the user types "cd", it might suggest common commands like "cd..", "cd /home", etc. The efficiency and relevance of these suggestions can significantly enhance user experience.

Trie Data Structure:

A Trie is a tree-like structure where each node represents a character.
Each path from the root to a leaf node forms a valid string (command).
Nodes store pointers to child nodes, allowing for efficient traversal based on prefixes.

#### Implementation
of Autocomplete with Trie:

Insert each prompt command into the Trie as you would insert words in a dictionary.
For each user input prefix, traverse the Trie to find matching commands.
Return all commands that match the given prefix.

Lookup Complexity: The time complexity for looking up a prefix in a Trie is O(L), where L is the length of the prefix. This is because you only need to follow the nodes corresponding to the characters in the prefix, and once you reach the end of the prefix, all commands that are children of this node will be the matches.

Storage Efficiency: Tries are more space-efficient for strings with common prefixes compared to hash maps or arrays. For example, if several commands share a prefix like "cd", only the shared prefix needs to be stored once in the Trie.

Comparison with Other Data Structures:

Hash Maps: While they offer O(1) average-time lookups, they don't handle prefixes as efficiently and require additional space for storing hashes.
Arrays: Searching through an array for a matching prefix is linear time (O(n)) in the worst case.

Edge Cases and Failures:

Ties can become sparse or unbalanced if most words are short, leading to increased memory usage but not affecting lookup complexity significantly.
If the Trie becomes very large, it might require optimization techniques like lazy deletion or compression of nodes with only one child.

#### Conclusion
Using a Trie for autocomplete systems is ideal when dealing with a significant number of commands with shared prefixes, as it provides efficient prefix-based lookups and better storage utilization compared to other data structures.

**Comparison:**

```
Command	Trie Node Structure
cd..	root -> c -> d -> .-> True
cd /home	root -> c -> d -> / -> h -> o -> m -> e -> True
ls -l	root -> l -> s -> - -> l -> True
```

**Code:**

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end_of_word = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end_of_word = True

    def search_prefix(self, prefix):
        node = self.root
        for char in prefix:
            if char not in node.children:
                return []
            node = node.children[char]
        return [word for word in self._collect_words(node)]

    def _collect_words(self, node):
        words = []
        if node.is_end_of_word:
            words.append("")
        for char, child_node in node.children.items():
            words.extend([char + suffix for suffix in self._collect_words(child_node)])
        return words
```

**Example input / output:**

```text
trie = Trie()
for command in data["commands"]:
    trie.insert(command)

# Example prefix lookups
print(trie.search_prefix("cd"))  # Output: ['cd..', 'cd /home']
print(trie.search_prefix("ls"))  # Output: ['ls -l']
```

**Data:**

```json
{
  "commands": ["cd..", "cd /home", "ls -l", "rm -rf"]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Start
5. TrieNode insert
6. Is char in children?
7. char node
8. New TrieNode
9. char node
10. is_end_of_word True
11. End insertion
12. Start
13. TrieNode search_prefix
14. Is char in children?
15. char node
16. collect_words
17. Return collected words

