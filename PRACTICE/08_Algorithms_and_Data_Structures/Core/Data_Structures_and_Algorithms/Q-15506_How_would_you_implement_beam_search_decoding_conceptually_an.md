---
type: "practice"
difficulty: "Deep"
---

# How would you implement beam search decoding conceptually, and what data structure holds the current candidate sequences?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Top-k and Sketches](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Top-k_and_Sketches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Beam search is a heuristic search algorithm used in natural language processing (NLP) and other fields where finding the optimal path through a large state space can be challenging. Conceptually, it works by maintaining a limited number of top candidate sequences at each step of the search process. This makes beam search more efficient than exhaustive search while still providing a good approximation of the best sequence.

Beam search decoding involves exploring multiple paths in parallel and pruning all but the best-scoring candidates at each step. The data structure that holds these current candidate sequences is typically an array or list, which we can extend to maintain the state across iterations.

To implement beam search, you need to define a scoring function for evaluating the quality of partial sequences. For example, in NLP tasks like text generation, this might be based on sequence likelihood given a model.

Here’s how we proceed step-by-step:

Initialization: Start with an initial set of candidate sequences (often just the start token).
Expansion: Generate all possible next tokens for each current sequence.
Scoring: Evaluate each new candidate sequence using your scoring function and keep only the top (k) based on their scores.
Termination: Repeat steps 2-3 until a stopping condition is met, such as reaching the maximum length or finding a valid terminal state.

Let's break this down further:

Step-by-Step Implementation

Define Initial State:

PYTHON
initial_sequences = [([start_token], score)]  # start with one sequence starting at the begin token

Expansion and Scoring: For each sequence, generate all possible next tokens and evaluate their scores.

PYTHON
for seq in initial_sequences:
    next_tokens_scores = model.predict_next_tokens(seq[0][-1])  # Predict the score of the next tokens given the last token in the current sequence
    expanded_sequences = [(seq[0] + [token], token_score * seq[1]) for token, token_score in next_tokens_scores]

Pruning: Keep only the top (k) sequences based on their scores.

PYTHON
initial_sequences = sorted(expanded_sequences, key=lambda x: -x[1])[:beam_width]  # Sort by score and keep top k

Check Termination Conditions: Check if any sequence has reached a terminal state or the maximum length is exceeded.

PYTHON
if all([seq[0][-1] == end_token for seq in initial_sequences]) or max_length_reached:
    return [seq[0] for seq in initial_sequences]
Data Structure

The data structure that holds these current candidate sequences can be an array or list, where each element is a tuple containing the sequence and its score. This allows us to efficiently manage and sort the candidates.

PYTHON
current_sequences = [
    ([start_token], 1.0),
    (['hello', 'world'], 0.85)
]

#### Common Mistakes/Follow-Ups
Mistake: Not properly pruning the sequences, leading to an unmanageable number of candidate sequences.
Mistake: Using too large a beam width, which can be computationally expensive and less efficient.

#### Follow-Up
Discussing trade-offs between model accuracy and computational cost.

Beam search is a powerful technique for generating high-quality sequences in NLP tasks but requires careful parameter tuning to balance exploration and exploitation effectively. Understanding the underlying principles will help you apply it more effectively in various contexts.

**Comparison:**

```
Step	Action
1	Initialize sequences with start token.
2	Expand current sequences by predicting next tokens and their scores.
3	Sort the expanded sequences by score and keep top k.
4	Check if any sequence has reached a terminal state or max length.
```

**Code:**

```python
def beam_search(model, start_token, end_token, max_length=20, beam_width=5):
    initial_sequences = [([start_token], 1.0)]

    for _ in range(max_length - 1):
        expanded_sequences = []

        for seq in initial_sequences:
            next_tokens_scores = model.predict_next_tokens(seq[0][-1])
            expanded_sequences.extend([(seq[0] + [token], token_score * seq[1]) for token, token_score in next_tokens_scores])

        # Prune top k sequences
        initial_sequences = sorted(expanded_sequences, key=lambda x: -x[1])[:beam_width]

        if all([seq[0][-1] == end_token for seq in initial_sequences]):
            return [seq[0] for seq in initial_sequences]

    return [seq[0] for seq in initial_sequences]
```

**Example input / output:**

```
Example input and output for beam_search function

model = DummyModel() # Placeholder model class start_token = '<START>' end_token = '<END>' beam_width = 5

sequences = beam_search(model, start_token, end_token, max_length=20, beam_width=5) print(sequences)

CODE
Copy

```plaintext
[['<START>', 'hello', 'world', '<END>'], ['<START>', 'good', 'morning', 'everyone', '<END>']]
```

**Data:**

```json
{
  "start_token": "<START>",
  "end_token": "<END>",
  "max_length": 20,
  "beam_width": 5,
  "sequences": [
    ["<START>", "hello", "world", "<END>"],
    ["<START>", "good", "morning", "everyone", "<END>"]
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is initial_sequences empty?
5. Initialize with start token
6. Generate next tokens for eachsequence
7. Predict scores and expandsequences
8. Sort by score and keep top k
9. Check if all sequences end ormax length reached?
10. Return current sequences asoutput

