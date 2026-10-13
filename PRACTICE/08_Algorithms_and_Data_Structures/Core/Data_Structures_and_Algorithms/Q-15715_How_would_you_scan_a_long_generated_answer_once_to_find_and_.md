---
type: "practice"
difficulty: "Basic"
---

# How would you scan a long generated answer once to find and collect all citation markers like [1] or [2], and what structure holds the found references?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Tries and Prefix Matching](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tries_and_Prefix_Matching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this problem, we need to understand that our goal is to traverse a string (representing a long generated answer) and identify all citation markers like [1], [2], etc., then collect these markers into an appropriate structure. The key steps involve iterating through the string and recognizing patterns that match our criteria.

#### Triaging the Question

The question asks us to:

Scan a text for specific patterns (citation markers).
Collect all occurrences of these patterns.
Store them in a structured format.

#### Assumptions

Citations are represented by square brackets containing numbers, e.g., [1], [2].
We need to ensure that our solution is efficient and works well with large texts.

#### First Principles Solution

We can solve this problem using basic string manipulation techniques. Here’s a step-by-step approach:

Initialize an empty list to hold the citation markers.
Iterate through the text: Use a loop or regular expression to find all matches of [ followed by any digit and then ].
Collect each match into our list.

Let's break this down further with an example:

#### Example

#### Cons
ider the following generated answer:

CODE
This is an example [1] paragraph where we want to find [2] citations like these.
Step-by-Step Solution
Initialize an empty list references to store the citation markers.
PYTHON
references = []

Iterate through each character in the text and check for patterns that match our criteria [digit].

Use a regular expression to find all matches:

The pattern we are looking for is: \[(\d+)\]
\[ and \] represent the literal square brackets.
(\d+) captures one or more digits.

Here’s how you can implement this in Python:

PYTHON
import re

def find_citations(text):
    references = []
    pattern = r'\[(\d+)\]'

    # Use regex to find all matches of citation markers
    for match in re.finditer(pattern, text):
        citations = match.group(1)  # Extract the number inside the brackets
        references.append(citations)

    return references

# Example usage:
text = "This is an example [1] paragraph where we want to find [2] citations like these."
print(find_citations(text))  # Output: ['1', '2']

#### Closing Summary
+ Common Mistakes/Follow-Ups

The solution involves using regular expressions to efficiently locate all citation markers in the text. The collected references are then stored in a list.

#### Common Mistakes

Forgetting to use proper escape sequences for square brackets.
Overlooking the possibility of multiple citations per line.
Not considering edge cases like [, ], or non-digit characters inside brackets.

#### Follow-Ups

Handling different citation formats (e.g., [1], [2], [3])
Optimizing performance with large texts
Adding error handling to manage invalid input

