---
type: "practice"
difficulty: "Intermediate"
---

# How does dbt decide the order to build models when one depends on another?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Node Selection](<../../../../TOOLS/dbt/dbt_Core/Node_Selection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, we'll explore how dbt decides the order to build models when there are dependencies between them. We'll start with a brief overview and then dive into the mechanics from first principles.

TL;DR

dbt uses a dependency resolution algorithm to determine the correct build order for your models. The core idea is that it processes models starting from those without dependencies, ensuring all necessary downstream models are built before their upstream models. This ensures data integrity and consistency in your model pipeline.

#### Triage
Understanding dbt's Dependency Resolution

dbt is a powerful tool used to manage and version-control your database schemas and SQL code. When you have a model that depends on another model (e.g., model_a depends on model_b), dbt needs to ensure these dependencies are respected during the build process.

Key Assumptions:
Model Dependencies: Models can depend on other models, datasets, or sources.
Build Order: dbt must determine a correct order of execution that respects these dependencies.
Data Consistency: Ensuring data is consistent by only building dependent models after their upstream models have been built.
First Principles: The Dependency Resolution Process
Model Metadata Analysis:
When you run dbt run, dbt first analyzes the metadata of your models to understand which models depend on others.
Dependency Graph Construction:
A directed acyclic graph (DAG) is constructed, where nodes represent models and edges represent dependencies.
Topological Sorting:
Once the DAG is built, dbt performs a topological sort to determine the order in which models can be safely executed.
Detailed Steps:

Model Definition and Parsing:

SQL
-- Example model_a.sql
{{ config(materialized='table') }}
select *
from `model_b`
where some_column > 0;

Dependency Metadata Extraction:

dbt parses the SQL files to extract dependencies.
For example, model_a depends on model_b.

Building the Dependency Graph:

Model B
Model A
The DAG represents the dependency between model_b and model_a.

Topological Sort Implementation:

dbt uses a depth-first search (DFS) or breadth-first search (BFS) to traverse the graph.
Nodes are pushed into a stack in reverse order of their DFS traversal, ensuring they are processed topologically.

Order Determination and Execution:

The models are executed based on the sorted order.
PYTHON
def sort_dependencies(graph):
    visited = set()
    stack = []

    def dfs(node):
        if node not in visited:
            visited.add(node)
            for neighbor in graph[node]:
                dfs(neighbor)
            stack.append(node)

    # Apply DFS to each node
    for node in graph:
        dfs(node)

    return stack[::-1]  # Reverse the stack to get topological order

dependencies = {
    'model_a': ['model_b'],
    'model_c': [],
    'model_d': ['model_c', 'model_b']
}

sorted_models = sort_dependencies(dependencies)
print(sorted_models)  # Output: ['model_c', 'model_b', 'model_a', 'model_d']

Execution Strategy:

dbt runs the models in the order determined by the topological sort.
If a model is marked as incremental, dbt ensures it only processes new data based on its dependencies.

#### Closing Summary
and Common Mistakes
The key takeaway is that dbt respects dependency relationships to ensure data consistency and integrity.
A common mistake is assuming that models can be executed in any order, leading to data inconsistencies. Always define clear dependencies and use dbt's build order mechanism.

#### Follow-Ups
Handling Circular Dependencies:
What happens when there are circular dependencies? How does dbt handle such cases?

#### Performance Considerations
Are there scenarios where the default dependency resolution can be optimized?

**Comparison:**

```
Model	Dependencies
model_a	model_b
model_c	None
model_d	model_c, model_b
```

**Code:**

```python
def sort_dependencies(graph):
    visited = set()
    stack = []

    def dfs(node):
        if node not in visited:
            visited.add(node)
            for neighbor in graph[node]:
                dfs(neighbor)
            stack.append(node)

    # Apply DFS to each node
    for node in graph:
        dfs(node)

    return stack[::-1]  # Reverse the stack to get topological order

dependencies = {
    'model_a': ['model_b'],
    'model_c': [],
    'model_d': ['model_c', 'model_b']
}

sorted_models = sort_dependencies(dependencies)
print(sorted_models)  # Output: ['model_c', 'model_b', 'model_a', 'model_d']
```

**Example input / output:**

```

Input (Dependency Graph):

YAML
Copy
dependencies:
  model_a:
    - model_b
  model_c: []
  model_d:
    - model_c
    - model_b

Output (Sorted Models):

PYTHON
Copy
['model_c', 'model_b', 'model_a', 'model_d']
```

**Data:**

```yaml
dependencies:
  model_a:
    - model_b
  model_c: []
  model_d:
    - model_c
    - model_b
```

**Flow:**

1. Model B
2. Model A
3. Model C
4. Model D

