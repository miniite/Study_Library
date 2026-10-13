---
type: "practice"
difficulty: "Basic"
---

# What data structure would you use to remember which prompts have already been sent so you can skip re-sending identical ones in a batch, and why?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Diff and LCS Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Diff_and_LCS_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To efficiently manage and check if a prompt has already been sent without re-sending duplicates, a HashSet is an ideal data structure to use. Here’s why:

HashSet Characteristics:

Unique Elements: A HashSet guarantees that no duplicate elements will be stored.
Fast Lookup: Insertions and lookups are very fast, typically O(1) on average.

Problem Context:

In a batch processing scenario where multiple prompts need to be sent, it's essential not to send the same prompt more than once to avoid redundant work and potential performance bottlenecks.

Reasoning From First Principles:

When you receive a new prompt to process, you first check if this prompt is already in your HashSet.
If the prompt isn’t found (meaning it’s unique), you add it to the HashSet and proceed with processing.
If the prompt is found, you skip its processing because it has already been sent.

#### Example
Implementation:

PYTHON
class PromptProcessor:
    def __init__(self):
        self.prompts_sent = set()

    def process_prompts(self, batch_of_prompts):
        processed_prompts = []
        for prompt in batch_of_prompts:
            if prompt not in self.prompts_sent:
                # Process the unique prompt here
                processed_prompts.append(prompt)
                # Add it to the HashSet after processing
                self.prompts_sent.add(prompt)
        return processed_prompts

#### Closing Summary
and Common Mistakes:
Using a HashSet ensures that you efficiently manage duplicate checks without adding unnecessary complexity.
A common mistake might be using a list instead, which would require O(n) time for each lookup and could lead to performance issues with large datasets.

