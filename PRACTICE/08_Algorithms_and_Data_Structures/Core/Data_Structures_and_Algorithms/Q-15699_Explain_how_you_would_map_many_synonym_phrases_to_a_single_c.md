---
type: "practice"
difficulty: "Basic"
---

# Explain how you would map many synonym phrases to a single canonical intent using a dictionary, and what its lookup complexity is.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Top-k and Sketches](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Top-k_and_Sketches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To map many synonym phrases to a single canonical intent, we can use a dictionary where each key represents a canonical phrase and the value maps to it. This approach ensures that all synonymous phrases are consistently treated as the same intent.

#### Triaging the Question

The question is asking for an efficient way to handle synonyms in natural language processing (NLP) tasks such as intent classification. The core of the problem lies in creating a mapping between multiple synonymous phrases and ensuring that they map to a single canonical phrase or "intent." The interviewer may be probing your understanding of data structures, specifically dictionaries, and how they can be used for efficient lookups.

#### Assumptions
Synonym Phrases: These are phrases that convey the same meaning as another phrase.
Canonical Phrase: A unique representation chosen to represent a set of synonymous phrases.
Efficiency: The solution should allow quick lookup and conversion from any synonym to the canonical phrase.

#### First-Principles Solution
Data Structure Selection:
We will use a dictionary (hash map) where each key is a canonical phrase, and the value points to that phrase.
Mapping Synonyms:
For each synonymous phrase, we can add it as a value in our dictionary, pointing to the same canonical phrase.

#### Example

Suppose we have the following phrases with their synonyms:

Canonical Phrase "Order Food" has synonyms "Place Order", "Get Food Ordered".
PYTHON
synonym_dict = {
    "Order Food": ["Place Order", "Get Food Ordered"],
    "Cancel Subscription": ["Unsubscribe", "Stop Service"]
}
Lookup Mechanism:
When a new phrase is input, we will use the dictionary to look up its canonical form.
The time complexity for lookup and insertion in an optimized hash map (like Python's dict) is (O(1)).
Full Solution:

To implement this solution, here’s a simple example using Python:

PYTHON
class IntentMapper:
    def __init__(self):
        self.synonym_dict = {}

    def add_synonyms(self, canonical_phrase, *synonyms):
        # Add the synonyms to the dictionary, all pointing to the same canonical phrase
        for syn in synonyms:
            if syn not in self.synonym_dict:  # Avoid duplicates
                self.synonym_dict[syn] = canonical_phrase

    def get_canonical_intent(self, intent):
        # Direct lookup of the intent from the dictionary
        return self.synonym_dict.get(intent)

# Example usage
mapper = IntentMapper()
mapper.add_synonyms("Order Food", "Place Order", "Get Food Ordered")
print(mapper.get_canonical_intent("Place Order"))  # Output: 'Order Food'

#### Closing Summary

By using a dictionary, we can efficiently map multiple synonymous phrases to their canonical form. This approach ensures that all variations of an intent are treated consistently. The lookup and insertion operations have an average time complexity of (O(1)), making it highly efficient for real-world applications.

#### Common Mistakes/Follow-Ups
Mistake: Not handling duplicates in the dictionary properly (e.g., adding a synonym that already exists).

#### Follow-Up
1: How would you handle phrases with multiple canonical forms?
You could use a set of canonical phrases for each input or implement a more complex data structure.

#### Follow-Up
2: What about phrases that don’t have any synonyms? Should they be added to the dictionary?
Yes, non-synonymous phrases should also be added as keys in the dictionary with their own canonical form.

