---
type: "practice"
difficulty: "Deep"
---

# You must build 'which of 20k saved alert rules match this incoming metric point' in real time; how do you invert query-per-document into a predicate index so matching is sub-linear in rules?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Inverted Indexes](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Inverted_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of efficiently querying which of 20k saved alert rules match an incoming metric point, we need to transform our approach from a per-document query to using predicate indices. This allows us to achieve sub-linear time complexity in matching the rules. Here’s a step-by-step breakdown of how we can implement this solution:

Triaging the Problem

The core challenge is to quickly determine which alert rules match an incoming metric point without performing 20k individual queries. The key idea here is to use predicate indices, which are precomputed and stored structures that allow us to rapidly filter out non-matching rules.

#### Assumptions

Each rule has conditions on specific metrics.
The metric point we receive includes values for the relevant metrics.
We need a solution that scales well as the number of rules grows.

#### First Principles Solution

Predicate Indexing:

For each alert rule, create an index based on its conditions.
Each condition can be broken down into individual predicates (e.g., metric_value > 100).
Store these predicates in a way that allows quick evaluation.

Index Construction:

For every predicate, maintain a data structure (such as a Trie or hash table) to quickly check if the incoming metric point satisfies the condition.
This step might involve pre-processing all rules and building these indices.

Query Optimization:

When a new metric point arrives, traverse the index structures to filter out rules that do not match.
This filtering can be done in parallel for efficiency.

Scalability Considerations:

Ensure the indexing mechanism is efficient enough to handle 20k+ rules and incoming metrics.
Optimize for both space and time, especially when dealing with high throughput scenarios.

#### Implementation
Example

Let's assume each alert rule has a condition like metric_value > 100 or metric_value < 200.

Step-by-Step Solution:

Index Construction:

For every rule, build an index based on its conditions.
PYTHON
class PredicateIndex:
    def __init__(self):
        self.index = {}

    def add_predicate(self, predicate):
        # Example: { "metric_value > 100": True, "metric_value < 200": True }
        self.index[predicate] = True

    def satisfies(self, metric_point):
        return all(self.index.get(pred) for pred in self.index)

Index Population:

Populate the index with predicates from each rule.
PYTHON
rules = [
    {"metric_value > 100": True, "metric_value < 200": False},
    # More rules...
]
indices = [PredicateIndex() for _ in range(len(rules))]
for i, rule in enumerate(rules):
    indices[i].add_predicate(rule)

Query Processing:

For each incoming metric point, query the indices.
PYTHON
def process_metric_point(metric_point):
    matching_rules = []
    for index, rule in zip(indices, rules):
        if index.satisfies(metric_point):
            matching_rules.append(rule)
    return matching_rules

#### Closing Summary
and Common Mistakes

The key takeaway is that by transforming the problem to use predicate indices, we can achieve sub-linear time complexity. This solution involves pre-processing all rules to build efficient data structures, which then allow for quick filtering during queries.

#### Common mistakes
include:

Failing to properly index each rule condition.
Not optimizing the traversal of indices for performance.
Ignoring potential edge cases where predicates are not straightforward comparisons.

