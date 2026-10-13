---
type: "practice"
difficulty: "Deep"
---

# You must invert 'which of 20k saved model-alert rules match this incoming prediction' in real time; how do you build a predicate index so matching is sub-linear in rules instead of query-per-rule?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Inverted Indexes](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Inverted_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To efficiently handle the problem of checking which of 20k saved model-alert rules match an incoming prediction in real-time, we need to build a predicate index that allows us to perform matching faster than linearly scanning each rule individually. This involves understanding the structure of the rules and leveraging data structures to achieve sub-linear time complexity.

#### Triage
Restating the Question

The question is asking how to preprocess 20k model-alert rules so that we can quickly determine which ones match a given prediction. The goal is to reduce the time complexity from (O(n)) (where (n = 20,000)) to something better.

#### Assumptions
and Key Considerations
Each rule has conditions that can be broken down into logical predicates.
We need to support efficient querying of these rules.
The solution should handle real-time data ingestion without significant latency.

#### First Principles Solution
Building a Predicate Index

Rule Parsing: Start by parsing each rule to understand its structure. For example, if the rule is "if temperature > 20 and humidity < 50", we can represent it as ((\text{temperature} > 20) \land (\text{humidity} < 50)).

Predicate Grouping: Group similar predicates together based on their type (e.g., numerical comparisons, string matches). This helps in reducing the number of distinct predicate types we need to handle.

Index Construction:

Use a Trie for string-based conditions.
Use a Binary Search Tree (BST) or Interval Tree for range-based conditions like temperature > 20 and humidity < 50.
For boolean combinations, use a tree structure where each node represents a logical operator.

Query Execution: When an incoming prediction is received, traverse the index to quickly identify matching rules. This can be done using techniques like prefix matching or range queries based on the type of predicate.

#### Example
Implementation

Here’s a simplified implementation outline in Python:

PYTHON
class PredicateNode:
    def __init__(self, condition=None):
        self.condition = condition  # e.g., (temperature > 20)
        self.children = {}  # maps operators to nodes
        self.rules = []  # list of rules that match this predicate

class PredicateTree:
    def __init__(self):
        self.root = PredicateNode()

    def add_rule(self, rule: dict):
        current_node = self.root
        for predicate in rule['predicates']:
            if predicate not in current_node.children:
                new_node = PredicateNode(predicate)
                current_node.children[predicate] = new_node
            current_node = current_node.children[predicate]
            current_node.rules.append(rule)

    def query(self, prediction: dict) -> list:
        results = []
        self._query_helper(self.root, prediction, [], results)
        return [rule['alert'] for rule in results]

    def _query_helper(self, node, prediction, path, results):
        if node.condition:
            # Check the current condition
            if eval(f"prediction['{node.condition}']"):
                path.append(node.condition)

        for key, child in node.children.items():
            if key in prediction:
                self._query_helper(child, prediction, path + [key], results)

        if len(path) == 0 and node.rules:
            results.extend(node.rules)

# Example usage
model_alert_rules = [
    {"predicates": ["temperature > 20", "humidity < 50"], "alert": "High Temperature Alert"},
    {"predicates": ["rain > 1", "windSpeed < 10"], "alert": "Rainy Weather Alert"}
]

tree = PredicateTree()
for rule in model_alert_rules:
    tree.add_rule(rule)

prediction = {"temperature": 25, "humidity": 40}
print(tree.query(prediction))  # Output: ["High Temperature Alert"]

#### Closing Summary
and Common Mistakes
Efficiency: Ensure that the index structure allows for efficient querying. Using data structures like Tries and Trees can significantly reduce query time.
Scalability: Consider how your solution will handle more complex rules or larger datasets in production.
Real-Time Performance: The index construction and query execution should be optimized to meet real-time requirements.

#### Common Mistakes
Not fully understanding the structure of the rules before indexing.
Overlooking the need for dynamic updates if rules change frequently.
Failing to consider edge cases, such as non-matching rules or complex nested conditions.

**Code:**

```python
class PredicateNode:
    def __init__(self, condition=None):
        self.condition = condition
        self.children = {}
        self.rules = []

class PredicateTree:
    def __init__(self):
        self.root = PredicateNode()

    def add_rule(self, rule: dict):
        current_node = self.root
        for predicate in rule['predicates']:
            if predicate not in current_node.children:
                new_node = PredicateNode(predicate)
                current_node.children[predicate] = new_node
            current_node = current_node.children[predicate]
            current_node.rules.append(rule)

    def query(self, prediction: dict) -> list:
        results = []
        self._query_helper(self.root, prediction, [], results)
        return [rule['alert'] for rule in results]

    def _query_helper(self, node, prediction, path, results):
        if node.condition:
            if eval(f"prediction['{node.condition}']"):
                path.append(node.condition)

        for key, child in node.children.items():
            if key in prediction:
                self._query_helper(child, prediction, path + [key], results)

        if len(path) == 0 and node.rules:
            results.extend(node.rules)
```

**Example input / output:**

```text
prediction = {"temperature": 25, "humidity": 40}
tree = PredicateTree()
for rule in model_alert_rules:
    tree.add_rule(rule)

print(tree.query(prediction))  # Output: ["High Temperature Alert"]
```

**Data:**

```json
[
    {
        "predicates": ["temperature > 20", "humidity < 50"],
        "alert": "High Temperature Alert"
    },
    {
        "predicates": ["rain > 1", "windSpeed < 10"],
        "alert": "Rainy Weather Alert"
    }
]
```

