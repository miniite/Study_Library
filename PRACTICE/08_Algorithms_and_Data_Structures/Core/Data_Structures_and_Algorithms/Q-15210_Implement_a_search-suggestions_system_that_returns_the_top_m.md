---
type: "practice"
difficulty: "Deep"
---

# Implement a search-suggestions system that returns the top matches as the user types each character.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tries and Prefix Matching](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tries_and_Prefix_Matching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a search-suggestions system, we need to design an efficient mechanism that provides relevant suggestions based on partial input as users type. The core challenge is to balance speed and accuracy of results, which can be achieved through a combination of data structures and algorithms. In this solution, we'll use a Trie (prefix tree) for efficient word storage and retrieval.

#### Triage

The question asks us to implement a search-suggestions system that returns the top matches as the user types each character. The interviewer is probing our understanding of data structures like tries and how they can be optimized for real-time searches, along with algorithmic optimizations such as heap-based sorting or other ranking mechanisms.

We'll assume:

The input consists of a list of words.
Each word is in a suitable string format (e.g., all lowercase).
The system should return the top k matches after each keystroke.
Words are scored based on relevance, which could be determined by factors like frequency or similarity.

#### First Principles Solution
Trie Construction: Build a Trie to store all words efficiently. Each node in the Trie represents a character and contains pointers to its children nodes.
Prefix Matching: As each character is typed, traverse the Trie to find all matching prefixes.
Ranking Mechanism: Sort these prefix matches based on relevance. We can use a simple ranking heuristic or incorporate more sophisticated techniques like Levenshtein distance for string similarity.
Top k Selection: Return the top k suggestions from the ranked list.
Detailed Steps
Trie Construction
Create a TrieNode structure with pointers to child nodes and an indicator for whether it marks the end of a word.
PYTHON
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end_of_word = False
Adding Words to Trie
Insert each word into the Trie, one character at a time.
PYTHON
def insert(word: str, root: TrieNode) -> None:
    node = root
    for char in word:
        if char not in node.children:
            node.children[char] = TrieNode()
        node = node.children[char]
    node.is_end_of_word = True
Prefix Matching
Traverse the Trie based on each keystroke to collect all matching words.
PYTHON
def find_prefix_matches(prefix: str, root: TrieNode) -> List[str]:
    current_node = root
    for char in prefix:
        if char not in current_node.children:
            return []
        current_node = current_node.children[char]
    matches = []
    collect_words(current_node, "", matches)
    return matches
def collect_words(node: TrieNode, word: str, matches: List[str]) -> None:
    if node.is_end_of_word:
        matches.append(word)
    for char, child in node.children.items():
        collect_words(child, word + char, matches)
Ranking Mechanism
Assign a score to each match based on relevance. For simplicity, we can use the length of the prefix as a heuristic.
PYTHON
def rank_matches(matches: List[str], prefix: str) -> List[Tuple[str, int]]:
    return sorted([(match, len(prefix)) for match in matches])
Top k Selection
Finally, select and return the top k ranked matches.
PYTHON
def get_top_suggestions(word: str, root: TrieNode, k: int) -> List[str]:
    matches = find_prefix_matches(word, root)
    scored_matches = rank_matches(matches, word)
    return [match for match, score in scored_matches[:k]]

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we've designed a Trie-based search-suggestions system that efficiently stores words and retrieves top matches based on user input. The key steps include building the Trie, prefix matching, ranking the results, and selecting the top k suggestions.

#### Common mistakes
to avoid:

Not properly handling edge cases where no matches are found.
Using suboptimal data structures or algorithms that significantly impact performance (e.g., linear search instead of Trie).
Overcomplicating the ranking mechanism without considering real-world relevance heuristics.

Potential follow-ups might include optimizing for memory usage, handling very large datasets, or incorporating more advanced ranking techniques like TF-IDF or cosine similarity.

