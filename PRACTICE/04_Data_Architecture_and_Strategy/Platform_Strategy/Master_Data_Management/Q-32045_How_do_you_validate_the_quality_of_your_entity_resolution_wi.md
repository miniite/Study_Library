---
type: "practice"
difficulty: "Expert"
---

# How do you validate the quality of your entity resolution without ground-truth labels — what evaluation strategy (blocking recall, pairwise precision, cluster metrics, human sampling) do you trust at scale?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Golden Records and Survivorship](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Golden_Records_and_Survivorship.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure high-quality entity resolution in a Master Data Management (MDM) system without ground-truth labels, we need to rely on evaluation strategies that can measure the quality of our resolution algorithm effectively. At an expert level, as a Data Engineer Architect/Principal, I would trust a combination of blocking recall, pairwise precision, cluster metrics, and human sampling for validation at scale.

Firstly, blocking recall is crucial because it ensures we have covered enough candidate pairs to make high-quality matches. High blocking recall indicates that most relevant pairs are being examined by the resolution algorithm.

Secondly, pairwise precision measures how accurate our matching is among all matched pairs. It’s a direct measure of the quality of individual matches and helps identify potential false positives or false negatives.

Thirdly, cluster metrics, such as purity, completeness, and compactness, provide insights into the overall structure and quality of the clusters formed by the resolution algorithm. Purity measures how homogeneous each cluster is with respect to its correct labels (if available), while completeness captures whether all relevant entities are captured within these clusters.

Lastly, human sampling offers a ground truth through manual review of a subset of resolved pairs. This method, though resource-intensive, provides the most accurate assessment and helps calibrate the other metrics by comparing their outcomes with actual human judgments.

Each of these strategies has its strengths and weaknesses, but together they provide a robust evaluation framework.

**Comparison:**

```
Metric	Description
Purity	Proportion of true clusters in the predicted clusters.
Completeness	Proportion of predicted clusters that are correct.
Compactness	Measures how tightly packed entities within a cluster are.

Note: This table is for illustrative purposes and actual implementation details may vary.
```

**Code:**

```python
# Example code for calculating cluster metrics (pseudo-code)
from sklearn.cluster import KMeans

def calculate_cluster_metrics(data):
    # Fit the data into clusters
    kmeans = KMeans(n_clusters=10).fit(data)
    labels = kmeans.labels_

    # Calculate purity, completeness, and compactness
    def purity(labels, true_labels):
        # Implementation of purity calculation

    def completeness(labels, true_labels):
        # Implementation of completeness calculation

    def compactness(cluster_centers, data):
        # Implementation of compactness calculation

    return {
        'purity': purity(true_labels, labels),
        'completeness': completeness(true_labels, labels),
        'compactness': compactness(kmeans.cluster_centers_, data)
    }
```

**Example input / output:**

```

Given the provided JSON data:

JSON
Copy
{
  "entities": [
    {"id": 1, "name": "Alice", "email": "alice@example.com"},
    {"id": 2, "name": "Bob", "email": "bob@sample.org"},
    {"id": 3, "name": "Charlie", "email": "charlie@example.com"}
  ],
  "clusters": [
    [1, 3],
    [2]
  ]
}

We can evaluate the clusters using:

Purity: If we know that Alice and Charlie are the same person (ground truth), both should be in the same cluster.
Completeness: Ensuring all entities are correctly grouped.
Compactness: Checking if each cluster is tightly packed based on similarity metrics.

Example calculations:

PYTHON
Copy
true_labels = [0, 1, 0]  # Assuming Alice and Charlie share a label, Bob does not

purity_score = purity(true_labels, labels)
print(f"Purity Score: {purity_score}")
```

**Data:**

```json
{
  "entities": [
    {"id": 1, "name": "Alice", "email": "alice@example.com"},
    {"id": 2, "name": "Bob", "email": "bob@sample.org"},
    {"id": 3, "name": "Charlie", "email": "charlie@example.com"}
  ],
  "clusters": [
    [1, 3],
    [2]
  ]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Start
6. Is blocking recall high?
7. Calculate pairwise precision
8. Are clusters pure, complete,and compact?
9. Cluster metrics met; resolutiongood
10. Improve blocking or resolutionalgorithm
11. Perform human sampling forground truth
12. Calibrate other metrics withhuman samples

