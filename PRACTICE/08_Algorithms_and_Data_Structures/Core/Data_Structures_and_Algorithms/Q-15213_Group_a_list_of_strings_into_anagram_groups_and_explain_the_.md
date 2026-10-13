---
type: "practice"
difficulty: "Intermediate"
---

# Group a list of strings into anagram groups and explain the canonical key you use.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Anagram Problems](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Anagram_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To group a list of strings into anagram groups, we need to identify words that contain the same characters in any order. A common approach is to sort the characters within each string and then group them by this sorted form. This way, all anagrams will have identical keys.

#### Triage

The question asks us to implement an algorithm to group strings into anagram groups using a canonical key. The interviewer likely wants to see your understanding of sorting, hashing, and data structures like dictionaries or hash maps. We'll assume the input is a list of strings and that the output should be a collection of lists where each sublist contains anagrams.

#### First-Principles Solution
Understand the Problem: Group words based on their sorted form.
Choose Data Structures: Use a dictionary to map from the canonical key (sorted string) to its corresponding list of anagrams.
Implement Sorting and Hashing:
Sort each string's characters.
Use this sorted version as a key in our dictionary.

Here is the step-by-step solution:

Initialize an empty dictionary.
Iterate through the input list of strings.
For each string, sort its characters to create a canonical key.
Add the original string to the list associated with that key in the dictionary.
Return the values from the dictionary as the result.

#### Example
Implementation
PYTHON
def group_anagrams(strs):
    anagram_groups = {}

    for s in strs:
        # Sort the characters and use it as a key
        sorted_s = ''.join(sorted(s))

        if sorted_s not in anagram_groups:
            anagram_groups[sorted_s] = []

        anagram_groups[sorted_s].append(s)

    return list(anagram_groups.values())

# Example input
input_strings = ["eat", "tea", "tan", "ate", "nat", "bat"]
output = group_anagrams(input_strings)

print(output)  # Expected output: [['eat', 'tea', 'ate'], ['tan', 'nat'], ['bat']]

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes
Forgetting to sort the characters properly, incorrectly implementing the logic for mapping strings to their anagram groups.

#### Follow-Ups
Discussing performance optimizations (e.g., using a more efficient sorting algorithm), handling edge cases like empty strings or duplicates in the input list.
Self-Check

Correctness + completeness confirmed.

