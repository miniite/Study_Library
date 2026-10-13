---
type: "practice"
difficulty: "Deep"
---

# How do you deal with high-cardinality label explosions blowing up per-series anomaly models?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Adaptive Thresholds](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Adaptive_Thresholds.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of high-cardinality label explosions in time series anomaly detection, it's crucial to understand that we're dealing with a scenario where a large number of unique labels (e.g., user IDs, device types) can cause an overwhelming increase in the number of per-series models needed. This problem is common in AIOps environments where you might have millions of devices or users generating time series data.

In essence, the core challenge is to manage and effectively monitor a vast number of time series without overfitting or becoming computationally unfeasible. We need to employ strategies that balance model complexity with performance while ensuring accurate anomaly detection.

#### Triage

The question asks how to handle high-cardinality label explosions in per-series anomaly models, which involves understanding the data distribution and choosing appropriate techniques for scaling and managing the models.

#### Assumptions
The system must handle a large number of unique labels.
Each label corresponds to its own time series.
Anomalies are detected independently for each time series.
Computational resources are limited, and we need efficient model management.

#### First-Principles Solution

To tackle this problem, we can break it down into several steps:

Dimensionality Reduction:

Hashing: Use hashing techniques to map high-cardinality labels to a lower-dimensional space while preserving the essential information.
Clustering: Group similar labels together using clustering algorithms like K-means or hierarchical clustering.

Feature Engineering:

Aggregation: Aggregate time series data at different levels (e.g., user groups, device types) and use these aggregated features as input to anomaly detection models.
Anomaly Detection Techniques: Utilize techniques that can handle high-dimensional inputs more efficiently, such as autoencoders or tree-based methods.

Model Management:

Batch Processing: Train the model in batches rather than on a per-series basis.
Online Learning: Implement online learning algorithms that update models incrementally as new data arrives.
Resource Allocation: Use resource-aware scheduling to manage computational resources effectively.

Anomaly Detection Framework:

Hybrid Models: Combine multiple anomaly detection techniques (e.g., statistical, machine learning) for better performance.
Ensemble Methods: Use ensemble methods like bagging or boosting to improve robustness and accuracy.

#### Closing Summary

To summarize, the key is to use a combination of dimensionality reduction, feature engineering, efficient model management, and advanced anomaly detection techniques to handle high-cardinality label explosions. By carefully designing our approach, we can maintain both performance and scalability in large-scale AIOps environments.

#### Common Mistakes/Follow-Ups
Overfitting: Failing to manage the complexity of individual models can lead to overfitting.
Insufficient Clustering: Poor clustering techniques might not adequately group similar labels, leading to suboptimal performance.
Inefficient Resource Management: Not properly managing computational resources can result in degraded performance or unavailability.

Would you like me to elaborate on any of these points or provide an example code snippet for a specific technique?

**Code:**

```python
# Example Python code using K-means clustering for dimensionality reduction
from sklearn.cluster import KMeans

def cluster_labels(labels, n_clusters=100):
    kmeans = KMeans(n_clusters=n_clusters)
    clustered_labels = kmeans.fit_predict(labels.reshape(-1, 1))
    return clustered_labels
```

**Example input / output:**

| Input Labels       | Clustered Labels |
|--------------------|------------------|
| user1              | 0                |
| user2              | 1                |
| deviceA            | 2                |
| deviceB            | 3                |
| sensorX            | 4                |
| sensorY            | 5                |

**Data:**

```json
{
  "labels": ["user1", "user2", "deviceA", "deviceB", "sensorX", "sensorY"],
  "clustered_labels": [0, 1, 2, 3, 4, 5]
}
```

