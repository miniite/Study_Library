---
type: "practice"
difficulty: "Deep"
---

# A feature-engineering DAG re-executes shared upstream transforms multiple times; how would you add memoization keyed on transform identity plus input hash, and how do you keep it correct when inputs change?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ LRU Cache Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/LRU_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of a feature-engineering Directed Acyclic Graph (DAG) re-executing shared upstream transforms multiple times, we can introduce memoization to cache the results based on the transform identity and input hash. This approach ensures that the DAG is optimized for performance while maintaining correctness when inputs change. Let's break down the solution step-by-step:

#### Triaging the Question

The core of this problem involves optimizing the execution of a feature-engineering pipeline by avoiding redundant computations. The key aspects are:

Memoization: Caching results to avoid re-computing them.
Transform Identity and Input Hash Keying: Ensuring that cached results are only used when both the transform identity and input hash match.

#### Assumptions
We have a DAG representing feature engineering processes, where each node represents a transformation or data processing step.
Each transformation has an identity (e.g., a unique identifier) and takes inputs which can change over time.
The goal is to cache results based on the combination of transform identity and input hash.
Step-by-Step Solution
Identify Transform Nodes: First, identify nodes in the DAG that represent transformations.
Input Hash Generation: For each transformation node, generate a unique hash from its inputs.
Memoization Storage: Use a cache to store results based on the combination of transform identity and input hash.
Execution Flow:
When a transformation is about to be executed, check if the result is already cached with the same transform identity and input hash.
If it’s cached, return the cached result.
If not, execute the transformation, cache the result, and then proceed.

#### First-Principles Solution
Node Identification: Assume we have a DAG represented as an adjacency list or graph structure.
Memoization Cache Initialization:
PYTHON
memoized_results = {}
Execution Function:
PYTHON
def execute_transform(transform_id, inputs):
    # Check if the result is already cached
    key = (transform_id, tuple(sorted(inputs)))
    if key in memoized_results:
        return memoized_results[key]

    # If not cached, compute and cache the result
    transformed_data = transform_function(transform_id, inputs)
    memoized_results[key] = transformed_data
    return transformed_data
Graph Traversal:
PYTHON
def traverse_dag(dag):
    for node in dag.nodes:
        if isinstance(node, TransformNode):  # Assuming nodes have a type attribute
            execute_transform(node.transform_id, node.inputs)

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing memoization with a cache key based on transform identity and input hash, we ensure that redundant computations are avoided while maintaining correctness. This approach is particularly useful in scenarios where the same transformation might be applied to different data sets multiple times.

#### Common Mistakes

Incorrect Cache Key: Failing to include both the transform identity and a unique input hash can lead to incorrect results.
Overwriting Cache: Ensuring that cached results are only overwritten when necessary is crucial to avoid stale data.
Further Discussion
Concurrency Issues: In a multi-threaded environment, ensure thread safety in cache access.
Evolution of Inputs: If the inputs change over time (e.g., new data), you may need to invalidate or update the cache accordingly.
Cache Size and Eviction Policies: Depending on the size of the DAG and the number of transformations, consider implementing eviction policies or a more sophisticated caching strategy.

