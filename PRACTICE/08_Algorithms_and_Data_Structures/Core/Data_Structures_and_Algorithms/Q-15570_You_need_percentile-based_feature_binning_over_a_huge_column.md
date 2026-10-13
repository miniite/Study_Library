---
type: "practice"
difficulty: "Deep"
---

# You need percentile-based feature binning over a huge column without sorting it all; how do approximate quantile sketches (GK or DDSketch) work and what error guarantee do they give for equal-frequency bins?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ External Memory Deduplication](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/External_Memory_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of percentile-based feature binning over a huge column without sorting it all, we need to understand how approximate quantile sketches like GK (Greenwald-Khanna) and DDSketch work. These algorithms allow us to estimate quantiles with theoretical guarantees on the error bounds, which is crucial for tasks where exact values are not feasible due to large data sizes.

The core idea behind these algorithms is to maintain a compact summary of the dataset that approximates the true quantiles efficiently. Both GK and DDSketch use sampling techniques to reduce the computational complexity while maintaining reasonable accuracy.

#### Triage
The question asks us to explain how approximate quantile sketches work, focusing on their internal mechanisms and error guarantees for equal-frequency bins (percentiles). We will assume a basic understanding of quantiles and binning but dive into the details of these algorithms from first principles.

#### First-principles solution

Understanding Quantiles: A k-th percentile divides a dataset such that approximately k% of values are below it.
GK Sketch (Greenwald-Khanna):
GK sketch uses a simple yet powerful approach to maintain an approximate quantile summary.
It works by inserting elements one at a time and maintaining two arrays: small and large.
For each element, if the current number of elements is less than 2k, it goes into small. Otherwise, it goes into large, and we remove the smallest element from small to keep the size consistent.
Error Guarantee: GK guarantees that the error in estimating the k-th quantile is bounded by ( O(1/\sqrt{k}) ). This means as the number of elements (k) increases, the error decreases, but it doesn’t guarantee exactness.
DDSketch:
DDSketch improves on GK by using a more sophisticated sampling mechanism and better handling of duplicate values.
It maintains an array of samples and uses techniques like reservoir sampling to ensure a representative subset of elements.

#### Closing summary
+ common mistakes/follow-ups: Understanding these algorithms is crucial for dealing with large datasets where exact quantile computation is impractical. Both GK and DDSketch provide a balance between accuracy and computational efficiency, making them suitable for big data applications. Common pitfalls include misinterpreting the error guarantees or not understanding how to properly configure parameters based on the specific requirements of the task.

**Comparison:**

| k       | Error Bound |
|---------|-------------|
| 10      | O(1/sqrt(10)) ≈ 34%   |
| 100     | O(1/sqrt(100)) = 10%  |

**Example input / output:**

```text
# Example input-output for GK sketch
Input: [10, 20, 30, 40, 50, 60]
Output:
- After first insertion (10): small=[10], large=[]
- After second insertion (20): small=[10, 20], large=[]
- After third insertion (30): small=[10, 20], large=[30]
- After fourth insertion (40): small=[10, 20], large=[30, 40]
- After fifth insertion (50): small=[10, 20], large=[30, 40, 50]
- After sixth insertion (60): small=[10, 20], large=[30, 40, 50, 60]

# For example, the approximate 75th percentile: ~50
```

**Flow:**

1. Yes
2. No
3. Insert Element
4. Is k < len(small) + len(large)?
5. Merge small and large, sort,return approximate quantile
6. Find insertion point in smallusing bisect
7. Remove smallest from small ifelement goes to large
8. Insert into large or replaceexisting value if necessary

