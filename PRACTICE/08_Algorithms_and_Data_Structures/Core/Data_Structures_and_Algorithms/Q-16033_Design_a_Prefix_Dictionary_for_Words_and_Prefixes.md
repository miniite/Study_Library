---
type: "practice"
difficulty: "Other"
---

# Design a Prefix Dictionary for Words and Prefixes

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Trie](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Trie.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design a data structure that stores a collection of lowercase words and can quickly answer two kinds of membership questions: whether an exact word was stored, and whether any stored word begins with a given prefix. This structure is the backbone of features like autocomplete and spell-checking.

Implement a class PrefixDictionary with the following operations:

PrefixDictionary() — initializes an empty dictionary.
void add(String word) — stores the string word into the dictionary.
boolean contains(String word) — returns true if the exact string word was previously added, and false otherwise.
boolean hasPrefix(String prefix) — returns true if there is at least one previously added word that starts with prefix, and false otherwise. (Note: an added word is always a prefix of itself.)

A prefix match does not require the prefix to be a full stored word — it only requires that some stored word begins with those characters.

#### Cons
traints

1 <= word.length, prefix.length <= 2000
word and prefix consist only of lowercase English letters (a–z).
At most 3 * 10^4 calls in total will be made to add, contains, and hasPrefix.

#### Example
s & test cases
TEXT
Input:
add("planet"); contains("planet"); contains("plan"); hasPrefix("plan"); add("plan"); contains("plan")
-> Output: (no return); true; false; true; (no return); true
Why: "planet" is stored, so contains("planet") is true; "plan" is only a prefix (not yet a stored word) so contains("plan") is false but hasPrefix("plan") is true; after add("plan"), contains("plan") becomes true.

Input: add("cat"); hasPrefix("dog") -> Output: (no return); false
Why: No stored word begins with "dog", so hasPrefix returns false.

Input: contains("apple") on an empty dictionary -> Output: false
Why: Nothing has been added, so no word can match.

Input: add("code"); contains("codex") -> Output: (no return); false
Why: "codex" was never stored; walking its characters runs past the stored word "code" into a missing node.

Input: add("aaaa"); hasPrefix("aaaa"); hasPrefix("aaaaa") -> Output: (no return); true; false
Why: The stored word equals the first prefix (a word is a prefix of itself) but is shorter than "aaaaa", so the longer prefix has no match.

#### Approach

Use a trie (prefix tree). Each node represents a position along some stored string and holds up to 26 child links (one per lowercase letter) plus an isWord flag marking where a full inserted word ends. add walks the word, creating any missing child nodes, and flags the terminal node as a word. Both queries reuse a single helper that walks from the root following the characters: if any needed child is missing it returns "not found." contains additionally requires the reached node's isWord flag to be set (the string must have been inserted as a whole word), whereas hasPrefix only needs the path to exist. Sharing common prefixes across words is what makes lookups depend on the query length rather than the number of stored words.

Complexity

Time: O(L) per operation, where L is the length of the word or prefix — each of add / contains / hasPrefix walks one character at a time down the tree. Space: O(total characters inserted) in the worst case, since each new character can create one node (with a 26-way branch factor for the array-based variants).

Python
PYTHON
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_word = False

class PrefixDictionary:
    def __init__(self):
        self.root = TrieNode()

    def add(self, word: str) -> None:
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_word = True

    def _find(self, s: str):
        node = self.root
        for ch in s:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node

    def contains(self, word: str) -> bool:
        node = self._find(word)
        return node is not None and node.is_word

    def has_prefix(self, prefix: str) -> bool:
        return self._find(prefix) is not None
JavaScript
JAVASCRIPT
class TrieNode {

#### cons
tructor() {
    this.children = {};
    this.isWord = false;
  }
}

class PrefixDictionary {

#### cons
tructor() {
    this.root = new TrieNode();
  }

  add(word) {
    let node = this.root;
    for (const ch of word) {
      if (!node.children[ch]) node.children[ch] = new TrieNode();
      node = node.children[ch];
    }
    node.isWord = true;
  }

  _find(s) {
    let node = this.root;
    for (const ch of s) {
      node = node.children[ch];
      if (!node) return null;
    }
    return node;
  }

  contains(word) {

#### cons
t node = this._find(word);
    return node !== null && node.isWord;
  }

  hasPrefix(prefix) {
    return this._find(prefix) !== null;
  }
}
Java
JAVA
class PrefixDictionary {
  private static class Node {
    Node[] children = new Node[26];
    boolean isWord = false;
  }

  private final Node root = new Node();

  public PrefixDictionary() {}

  public void add(String word) {
    Node node = root;
    for (char ch : word.toCharArray()) {
      int i = ch - 'a';
      if (node.children[i] == null)
        node.children[i] = new Node();
      node = node.children[i];
    }
    node.isWord = true;
  }

  public boolean contains(String word) {
    Node node = find(word);
    return node != null && node.isWord;
  }

  public boolean hasPrefix(String prefix) {
    return find(prefix) != null;
  }

  private Node find(String s) {
    Node node = root;
    for (char ch : s.toCharArray()) {
      int i = ch - 'a';
      if (node.children[i] == null)
        return null;
      node = node.children[i];
    }
    return node;
  }
}
C++
CPP
#include <string>
#include <array>
using namespace std;

class PrefixDictionary {
 public:
  PrefixDictionary() : root(new Node()) {}

  void add(const string& word) {
    Node* node = root;
    for (char ch : word) {
      int i = ch - 'a';
      if (!node->children[i])
        node->children[i] = new Node();
      node = node->children[i];
    }
    node->isWord = true;
  }

  bool contains(const string& word) {
    Node* node = find(word);
    return node && node->isWord;
  }

  bool hasPrefix(const string& prefix) {
    return find(prefix) != nullptr;
  }

 private:
  struct Node {
    array<Node*, 26> children{};  // value-initialized to nullptr
    bool isWord = false;
  };
  Node* root;

  Node* find(const string& s) {
    Node* node = root;
    for (char ch : s) {
      int i = ch - 'a';
      if (!node->children[i])
        return nullptr;
      node = node->children[i];
    }
    return node;
  }
};
C
C
#include <stdlib.h>
#include <stdbool.h>

typedef struct TrieNode {
    struct TrieNode* children[26];
    bool isWord;
} TrieNode;

static TrieNode* newNode(void) {
    return calloc(1, sizeof(TrieNode));  // children NULL, isWord false
}

typedef struct {
    TrieNode* root;
} PrefixDictionary;

PrefixDictionary* prefixDictionaryCreate(void) {
    PrefixDictionary* pd = malloc(sizeof(PrefixDictionary));
    pd->root = newNode();
    return pd;
}

void prefixDictionaryAdd(PrefixDictionary* pd, const char* word) {
    TrieNode* node = pd->root;
    for (int i = 0; word[i] != '\0'; i++) {
        int c = word[i] - 'a';
        if (node->children[c] == NULL)
            node->children[c] = newNode();
        node = node->children[c];
    }
    node->isWord = true;
}

static TrieNode* findNode(PrefixDictionary* pd, const char* s) {
    TrieNode* node = pd->root;
    for (int i = 0; s[i] != '\0'; i++) {
        int c = s[i] - 'a';
        if (node->children[c] == NULL)
            return NULL;
        node = node->children[c];
    }
    return node;
}

bool prefixDictionaryContains(PrefixDictionary* pd, const char* word) {
    TrieNode* node = findNode(pd, word);
    return node != NULL && node->isWord;
}

bool prefixDictionaryHasPrefix(PrefixDictionary* pd, const char* prefix) {
    return findNode(pd, prefix) != NULL;
}

#### Follow-up
questions
Add a remove(word) operation that deletes a stored word and prunes any nodes that are no longer part of any word.
Support search with a single-character wildcard (e.g. .) that can match any letter at that position.
Return all stored words that start with a given prefix (autocomplete), ideally in sorted order.
Extend the alphabet beyond lowercase a–z (Unicode / mixed case) and discuss how the array-vs-hashmap child representation trade-off changes.

