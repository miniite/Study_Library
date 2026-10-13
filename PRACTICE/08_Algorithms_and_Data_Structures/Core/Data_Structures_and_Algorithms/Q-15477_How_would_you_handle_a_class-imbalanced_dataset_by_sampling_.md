---
type: "practice"
difficulty: "Intermediate"
---

# How would you handle a class-imbalanced dataset by sampling, and what data structure lets you pick from each class in O(1)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ External Memory Deduplication](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/External_Memory_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In handling a class-imbalanced dataset, one common approach is to use resampling techniques such as oversampling or undersampling to balance out the class distribution. For this problem, we'll focus on sampling with the goal of achieving an equal number of samples from each class. The key data structure that allows us to efficiently pick samples in constant time (O(1)) is a hash table (or dictionary).

To tackle this question effectively, let's first clarify what the interviewer is probing: understanding of resampling techniques, proficiency with hash tables, and the ability to reason about time complexity.

#### Triaging the Question

The core of the problem revolves around balancing class representation in an imbalanced dataset. We need a data structure that allows us to quickly select samples from each class so that we can achieve a more balanced dataset. The requirement for (O(1)) access is crucial, as it guarantees efficient sampling.

#### First-Principles Solution
Step 1: Understanding the Problem

Given an imbalanced dataset with multiple classes, our goal is to balance the number of samples from each class by either oversampling (increasing the number of samples in underrepresented classes) or undersampling (decreasing the number of samples in overrepresented classes). We need a data structure that supports efficient random access to samples.

Step 2: Choosing the Data Structure

A hash table is an ideal choice for this problem because it allows us to store and retrieve elements with constant time complexity (O(1)). Each class will have its own collection of indices (or sample IDs) stored in a separate array or list, which can be accessed via a hash function. This way, we can quickly pick samples from any class.

Step 3: Implementation

Let's walk through the implementation step by step:

Initialization: Create a dictionary where each key is a class label and its value is an empty list that will store the indices of samples belonging to that class.
Populate with Indices: Traverse the dataset and for each sample, add its index (or ID) to the corresponding class’s list in the hash table.
Sampling: To pick a random sample from a specific class, simply generate a random index within the range of indices stored in the class's list.

Here is a Python implementation:

PYTHON
import random

class SampleBalancer:
    def __init__(self):
        self.class_indices = {}

    def add_samples(self, samples, labels):
        for i, label in enumerate(labels):
            if label not in self.class_indices:
                self.class_indices[label] = []
            self.class_indices[label].append(i)

    def sample_from_class(self, class_label):
        indices = self.class_indices.get(class_label)
        return indices[random.randint(0, len(indices) - 1)] if indices else None

# Example usage
samples = [1, 2, 3, 4, 5, 6, 7]
labels = ['A', 'B', 'A', 'C', 'B', 'C', 'A']
balancer = SampleBalancer()
balancer.add_samples(samples, labels)

print(balancer.sample_from_class('A'))  # Random index from class A

#### Closing Summary
and Common Mistakes/Follow-Ups

Using a hash table ensures efficient random access to samples in constant time. However, there are some common pitfalls:

Hash Collisions: While rare with good hashing functions, collisions can lead to inefficiencies.
Memory Usage: Storing all indices for each class can consume significant memory.

For follow-ups, the interviewer might ask about:

How to handle large datasets that don't fit into memory (e.g., streaming data).
What if we want to sample without replacement?
How this approach scales with more classes and samples?

