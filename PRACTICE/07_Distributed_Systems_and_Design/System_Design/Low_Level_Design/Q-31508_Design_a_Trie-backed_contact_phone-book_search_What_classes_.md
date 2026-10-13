---
type: "practice"
difficulty: "Intermediate"
---

# Design a Trie-backed contact/phone-book search. What classes and how does prefix search work?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Threaded Comments Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Threaded_Comments_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

For designing a Trie-backed contact/phone book, we will create a TrieNode class to represent each node in the Trie structure. The Trie class will manage these nodes and provide methods for inserting contacts and searching based on prefixes. Prefix search works by traversing the Trie from the root until the prefix is found or exhausted.

#### Triaging the Question

The question asks us to design a contact/phone book using a Trie data structure, specifically focusing on how it supports efficient prefix searches. The interviewer is probing our understanding of Trie data structures and their application in searching and storing strings like phone numbers and names. We need to assume that we are working with a collection of contacts where each contact has a name or number (e.g., "John Doe" and "123-456-7890").

#### First Principles Solution
Step 1: Define the TrieNode Class

A TrieNode will store information about the current character and whether it's the end of a word. Each node can also point to child nodes.

PYTHON
class TrieNode:
    def __init__(self):
        self.children = {}  # A dictionary of children nodes, key is char, value is TrieNode
        self.is_end_of_word = False  # Whether this node represents the end of a word
Step 2: Define the Trie Class

The Trie class will handle insertion and search operations.

PYTHON
class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, contact: str) -> None:
        node = self.root
        for char in contact:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end_of_word = True

    def search_prefix(self, prefix: str) -> bool:
        node = self._search_node(prefix)
        return node is not None and node.is_end_of_word
Step 3: Implement Search Operations

The search_prefix method searches for a given prefix and returns whether it exists as the start of any word.

PYTHON
def _search_node(self, prefix: str) -> TrieNode:
    node = self.root
    for char in prefix:
        if char not in node.children:
            return None
        node = node.children[char]
    return node

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing the Trie class, we can efficiently store and search through contacts. The Trie structure ensures that each insertion and search operation operates in O(m) time, where m is the length of the word or prefix.

#### Common mistakes
include:

Forgetting to check if a character exists in a node's children.
Mismanaging is_end_of_word flags, which can lead to incorrect results when searching for prefixes.
Not handling edge cases such as empty strings or invalid characters.

#### Follow-up
questions might explore optimizations like compression techniques (e.g., sharing common suffixes) or how this design scales with a large number of contacts.

Triangulation Summary

Tries are particularly useful for prefix searches because they allow for efficient traversal based on the prefix. By structuring our Trie correctly, we can achieve both insertion and search operations in linear time relative to the length of the input string.

