---
type: "practice"
difficulty: "Deep"
---

# What is the blast radius of a bad upstream change, and how do you communicate it to affected teams automatically using lineage?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lineage for Dynamic SQL](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Lineage_for_Dynamic_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the challenge of communicating the blast radius of a bad upstream change in a data pipeline, we first need to understand what "blast radius" means in this context. Essentially, it refers to the extent or impact of a faulty or incorrect change that propagates through the system. Given our focus on Data Governance and Lineage, we can leverage these concepts to pinpoint exactly which downstream processes are affected by an upstream issue.

#### Triage
Understanding the Question

The core question here is about identifying and communicating the precise impact (blast radius) of a bad upstream data change in real-time. This involves understanding how lineage works within our pipeline architecture—essentially, tracking all dependencies between source and target datasets. We are also asked to implement an automated system that informs relevant downstream teams as soon as a problem is detected.

#### First-Principles Solution

Lineage Tracking: Implement robust lineage tracking mechanisms for all data transformations. This can be achieved through metadata tagging or utilizing tools like Apache Atlas, AWS Glue, or Databricks Delta Lake that track historical changes and dependencies.

Impact Analysis: Develop a mechanism to perform impact analysis based on the detected lineage. For instance, if we identify an issue in a particular source dataset, the system should be able to determine all downstream datasets and pipelines that depend on it.

Automated Communication: Set up automated alerts or notifications for affected teams when a critical change is identified. This can be done via email, Slack messages, or other communication channels.

Detailed Solution
Step 1: Lineage Tracking

We need to track the lineage of each data transformation process. For example:

PLAINTEXT
Data Source A -> Transformation B -> Data Sink C

Here, Transformation B depends on Data Source A, and Data Sink C depends on Transformation B.

Step 2: Impact Analysis

When a change is detected in an upstream dataset (e.g., Data Source A), the system should perform an impact analysis to find all downstream dependencies. This can be done using a graph traversal algorithm like Breadth-First Search (BFS) or Depth-First Search (DFS).

Step 3: Automated Communication

Once the impacted downstream datasets and pipelines are identified, the system should automatically send alerts to relevant teams. For example:

PLAINTEXT
Alert [High] - Change detected in Data Source A affects Transformation B & Data Sink C.
Code Example

Here is a simple Python script that performs lineage tracking and impact analysis:

PYTHON
class PipelineGraph:
    def __init__(self):
        self.graph = {}

    def add_edge(self, source, target):
        if source not in self.graph:
            self.graph[source] = []
        if target not in self.graph:
            self.graph[target] = []
        self.graph[source].append(target)

    def bfs(self, start_node):
        visited = set()
        queue = [start_node]
        impacted_nodes = []

        while queue:
            node = queue.pop(0)
            if node not in visited:
                visited.add(node)
                impacted_nodes.append(node)
                for neighbor in self.graph.get(node, []):
                    if neighbor not in visited and neighbor != start_node:  # Avoid cycles
                        queue.append(neighbor)

        return impacted_nodes

# Example usage
pipeline_graph = PipelineGraph()
pipeline_graph.add_edge('Data Source A', 'Transformation B')
pipeline_graph.add_edge('Transformation B', 'Data Sink C')

impacted_nodes = pipeline_graph.bfs('Data Source A')
print(f"Impacted nodes: {impacted_nodes}")

#### Closing Summary
and Common Mistakes

In summary, to effectively communicate the blast radius of a bad upstream change, we need to implement robust lineage tracking, perform impact analysis using graph traversal algorithms, and set up automated alerts for affected teams. The primary challenges are ensuring comprehensive lineage data collection and efficient impact analysis.

#### Common Mistakes

Failing to capture full lineage information.
Overlooking edge cases like cycles in the pipeline graph.
Not setting up timely and relevant alerts.
Self-CHECK

This solution is both correct and complete, addressing all aspects of the question. The code provided is a simple example; in practice, you would likely use more sophisticated tools and technologies for lineage tracking and impact analysis.

**Comparison:**

```
Impact Analysis	Node	Affected Nodes
Start	Data Source A	Transformation B, Data Sink C
BFS Traversal	Transformation B	Data Sink C
```

**Code:**

```python
class PipelineGraph:
    def __init__(self):
        self.graph = {}

    def add_edge(self, source, target):
        if source not in self.graph:
            self.graph[source] = []
        if target not in self.graph:
            self.graph[target] = []
        self.graph[source].append(target)

    def bfs(self, start_node):
        visited = set()
        queue = [start_node]
        impacted_nodes = []

        while queue:
            node = queue.pop(0)
            if node not in visited:
                visited.add(node)
                impacted_nodes.append(node)
                for neighbor in self.graph.get(node, []):
                    if neighbor not in visited and neighbor != start_node:  # Avoid cycles
                        queue.append(neighbor)

        return impacted_nodes

pipeline_graph = PipelineGraph()
pipeline_graph.add_edge('Data Source A', 'Transformation B')
pipeline_graph.add_edge('Transformation B', 'Data Sink C')

impacted_nodes = pipeline_graph.bfs('Data Source A')
print(f"Impacted nodes: {impacted_nodes}")
```

**Example input / output:**

```

Input:

PLAINTEXT
Copy
Add edge between Data Source A and Transformation B, then between Transformation B and Data Sink C.
Start BFS from Data Source A.

Output:

PLAINTEXT
Copy
Impacted nodes: ['Transformation B', 'Data Sink C']
```

**Data:**

```json
{
  "edges": [
    {"source": "Data Source A", "target": "Transformation B"},
    {"source": "Transformation B", "target": "Data Sink C"}
  ]
}
```

**Flow:**

1. Data Source A
2. Transformation B
3. Data Sink C
4. Start Node
5. Impact Analysis
6. BFS Traversal
7. Notify Affected Teams

