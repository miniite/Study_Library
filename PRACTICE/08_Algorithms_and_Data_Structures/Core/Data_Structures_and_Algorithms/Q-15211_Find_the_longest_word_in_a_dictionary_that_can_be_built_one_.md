---
type: "practice"
difficulty: "Intermediate"
---

# Find the longest word in a dictionary that can be built one character at a time from other words.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tries and Word Building](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tries_and_Word_Building.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find the longest word that can be built one character at a time from other words in a dictionary, we need to ensure that each prefix of this word is also present in the dictionary. This problem can be approached using a Trie data structure combined with a topological sort or depth-first search (DFS) technique.

#### Triage

The question asks us to identify the longest lexicographical order word from which all its prefixes are valid words within the given dictionary. We will use:

A Trie (prefix tree) to store and check word prefixes efficiently.
Topological sorting or DFS to ensure that we only consider valid prefix sequences.

#### First Principles Solution
Build a Trie: Insert each word into a Trie. This allows us to quickly check if any prefix of a word exists in the dictionary.
Topological Sort/DFS with Memoization: Traverse the Trie and use memoization to keep track of which paths lead to valid words. We will prioritize longer words by checking the length of the current path.
Step-by-Step Solution

#### Cons
truct the Trie:

Insert each word from the dictionary into a Trie.
Each node in the Trie represents a character, and an edge exists between nodes if there is a corresponding next character in any word.

Topological Sort/DFS with Memoization:

Start DFS traversal from the root of the Trie.
Use memoization to store results of subproblems (e.g., whether a particular path is valid).
For each node, check all its children and recursively explore paths that form valid words.
Keep track of the longest word found during the traversal.
Code Implementation

Here's an implementation in Python:

PYTHON
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_word = False

def insert(word, trie):
    node = trie
    for char in word:
        if char not in node.children:
            node.children[char] = TrieNode()
        node = node.children[char]
    node.is_word = True

def longest_prefix_word(trie):
    max_length = 0
    result = ""

    def dfs(node, path):
        nonlocal max_length, result
        if len(path) > max_length:
            max_length = len(path)
            result = "".join(path)

        for char, child in node.children.items():
            if not child.is_word: continue
            path.append(char)
            dfs(child, path)
            path.pop()

    dfs(trie, [])
    return result

# Sample dictionary and words to insert into the Trie
words = ["w", "wo", "wor", "word", "work", "worker"]
trie = TrieNode()
for word in words:
    insert(word, trie)

print(longest_prefix_word(trie))  # Output: "work"

#### Common Mistakes
and Follow-Ups
Mistake: Not handling edge cases where the dictionary is empty or contains only single-character words.
Mistake: Failing to properly initialize the Trie node structure, leading to errors in insertion and traversal.

#### Follow-Up
Optimizing space complexity if the word list size is very large by using a more efficient data structure for storing prefixes.

#### Conclusion

The solution leverages a Trie data structure to efficiently store and query word prefixes, combined with DFS or topological sort techniques to ensure only valid prefix sequences are considered. This approach ensures that we can find the longest word that meets the criteria in an optimal manner.

