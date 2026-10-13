---
type: "practice"
difficulty: "Intermediate"
---

# What is the time complexity of trie insert and search for a word of length L?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stack-based Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Stack-based_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To analyze the time complexity of Trie (prefix tree) operations, specifically insert and search, we need to consider how these operations are implemented and their performance relative to the input size. For this discussion, let's assume a Trie where each node has a fixed number of children (usually equal to the alphabet size), denoted as ( k ).

#### Triage
What the Question is Asking

The question is probing your understanding of algorithmic efficiency in string data structures like tries. The goal is to identify how operations scale with input length and structure.

#### First-Principles Solution

Let's break down both insert and search operations step by step.

Insert Operation:
Node Traversal: For each character in the word, we traverse a path from the root to potentially a new node.
Branch Creation or Update: At each level, we might either create a new node (if it doesn't exist) or update an existing one with the next character.

Time Complexity for Insert:

Each insertion involves traversing ( L ) nodes and branches.
The number of operations required per node is constant (creating/setting a pointer).
Therefore, the overall time complexity is O(L), where ( L ) is the length of the word.
Search Operation:
Node Traversal: Similar to insert, we traverse from the root node, character by character.
Path Validation: Each traversal checks whether each character leads to a valid node in the Trie.

Time Complexity for Search:

Like the insert operation, searching also involves traversing ( L ) nodes and branches.
The validation step at each node is constant time (checking if a pointer exists).
Therefore, the overall time complexity is O(L), where ( L ) is the length of the word.

#### Closing Summary

Both Trie insertion and search operations have linear time complexity with respect to the input word length. This means that the performance scales directly proportional to the number of characters in the word being processed. The constant factor involved depends on the implementation details such as how nodes are managed, but typically remains manageable for practical purposes.

#### Common Mistakes/Follow-Ups
Ignoring Constant Factors: Some may mistakenly consider additional overhead due to node creation or pointer updates, which can be minimal.
Misunderstanding Trie Structure: Not recognizing that each character in the word is processed sequentially and independently affects the overall time complexity.
Failing to Consider Deletions: While not directly asked, understanding how deletions might impact the same operations can be a valuable follow-up.

**Comparison:**

```
Operation	Insert	Search
Time Complexity	O(L)	O(L)
Space Complexity	O(N * L) (N is the number of words in the Trie, L is the average length of the words)	O(1) + auxiliary space for recursive call stack
```

**Code:**

```python
# Example Python function for Trie Insertion

class TrieNode:
    def __init__(self):
        self.children = [None] * 26  # Assuming lowercase English alphabet
        self.is_end_of_word = False

def insert(root, word):
    current_node = root
    for char in word:
        index = ord(char) - ord('a')
        if not current_node.children[index]:
            current_node.children[index] = TrieNode()
        current_node = current_node.children[index]
    current_node.is_end_of_word = True

# Example Python function for Trie Search

def search(root, word):
    current_node = root
    for char in word:
        index = ord(char) - ord('a')
        if not current_node.children[index]:
            return False
        current_node = current_node.children[index]
    return current_node.is_end_of_word
```

**Example input / output:**

```text
Input: insert("hello", root)
Output: None (root remains unchanged but now has a node for 'h', 'he', 'hel', 'hell', and 'hello')

Input: search("world", root)
Output: False

Input: search("hello", root)
Output: True
```

**Data:**

```json
{
  "root": {
    "children": [
      null, // 'a'
      null, // 'b'
      null, // ...
      { "is_end_of_word": false }
    ],
    "is_end_of_word": false
  }
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. root --> a --> b --> c --> d --> e --> f(is_end_of_word);
4. subgraph Word "hello"
5. h --> e --> l[2 nodes]
6. l --> l[1 node]
7. end

