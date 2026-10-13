---
type: "practice"
difficulty: "Deep"
---

# Your training data pipeline must reservoir-sample k rows from an unbounded stream to build a balanced validation set; prove why the algorithm gives uniform probability and how you extend it to weighted sampling.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ External Memory Deduplication](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/External_Memory_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of reservoir-sampling (k) rows from an unbounded stream to create a balanced validation set, we'll delve into the algorithm's fundamental principles and its extension to weighted sampling. The core idea is to ensure that each element in the stream has an equal probability of being selected for the sample.

#### Triaging the Question

The question asks us to design an algorithm that can reservoir-sample (k) rows from an unbounded stream, ensuring uniform probability for all elements. Additionally, we need to extend this concept to weighted sampling where different elements have varying probabilities of selection based on their weights.

Key Assumptions:
The stream is infinite and non-replayable.
We want a sample that reflects the overall distribution of the data.
The algorithm must be efficient in terms of time complexity, ideally constant for each new element added to the stream.

#### First-Principles Solution

Let's start by understanding the reservoir-sampling algorithm. The basic idea is to maintain a "reservoir" of size (k). Initially, fill the reservoir with the first (k) elements of the stream. For every subsequent element (i > k), there's a probability (\frac{k}{i}) that it replaces one of the elements in the reservoir.

Step-by-Step Proof
Initial Filling: Fill the reservoir with the first (k) elements.
Reservoir Maintenance:
For each element (x_i) at position (i > k), generate a random number (r) from ([0, i)).
If (r < k), replace the element at position (r) in the reservoir with (x_i).

This approach ensures that every element has an equal probability of being included in the sample. To prove uniform probability:

#### Cons
ider any element (i) and its position in the stream.
The probability that (i) is chosen initially (if (i \leq k)) is (\frac{1}{k}).
For elements beyond (k), the probability of replacement at step (i) is (\frac{k}{i}).
Extending to Weighted Sampling

To extend this to weighted sampling, we need to modify the algorithm to account for weights. Weights can be represented as a factor that influences the selection probability.

Initialization: Similar to the unweighted case, fill the reservoir with the first (k) elements.
Reservoir Maintenance:
For each element (x_i) at position (i > k), generate a weighted random number based on its weight and the weights of the current elements in the reservoir.
Replace one of the elements in the reservoir if it is chosen by the weighted random number.
Detailed Algorithm
PYTHON
def weighted_reservoir_sampling(stream, k):
    reservoir = []
    total_weight = 0

    for i, (element, weight) in enumerate(stream):
        # Initialize reservoir with first k elements
        if len(reservoir) < k:
            reservoir.append((element, weight))
            total_weight += weight
            continue

        # Calculate probability of selection based on weights
        r = random.uniform(0, total_weight)

        for j, (elem, elem_wt) in enumerate(reservoir):
            if r < elem_wt:
                reservoir[j] = (element, weight)
                break
            r -= elem_wt

    return [e for e, _ in reservoir]

#### Closing Summary
and Common Mistakes/Follow-Ups
Uniform Probability: The algorithm ensures uniform probability by maintaining a constant selection rate throughout the stream.
Weighted Sampling: By incorporating weights into the random number generation process, we can adjust the sampling distribution.

#### Common Mistakes

Failing to account for the correct replacement probabilities.
Not understanding how weights influence the selection process.

#### Conclusion

Reservoir-sampling provides a robust solution for creating balanced validation sets from unbounded streams. The extension to weighted sampling ensures that elements are selected according to their importance, maintaining the integrity of the sample distribution.

**Comparison:**

| Element ID | Weight |
|------------|--------|
| 1          | 2.4    |
| 2          | 7.8    |
| 3          | 5.6    |
| ...        | ...    |

**Code:**

```python
import random

def reservoir_sampling(stream, k):
    reservoir = []

    for i, element in enumerate(stream):
        if len(reservoir) < k:
            reservoir.append(element)
        else:
            j = random.randint(0, i)
            if j < k:
                reservoir[j] = element

    return reservoir
```

**Example input / output:**

```text
# Example input and output

stream = [{"id": i, "weight": random.uniform(1, 10)} for i in range(1000)]
k = 10
sampled_stream = weighted_reservoir_sampling(stream, k)
print(sampled_stream)
```

**Data:**

```json
{
  "stream": [
    {"id": 1, "value": 20},
    {"id": 2, "value": 30},
    {"id": 3, "value": 40},
    ...
  ],
  "k": 5
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Initialize Reservoir
6. Is i <= k?
7. Add element to reservoir
8. Generate random number rfrom 0 to total weight
9. Does r < current_weight?
10. Replace element in reservoirwith new one
11. Continue to next element

