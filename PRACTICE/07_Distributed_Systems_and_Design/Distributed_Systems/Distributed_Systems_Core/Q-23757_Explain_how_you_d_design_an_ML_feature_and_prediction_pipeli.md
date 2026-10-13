---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd design an ML feature and prediction pipeline for a real-time fraud model that must fail closed (block the transaction) rather than fail open when a feature dependency is down, while a recommendation model on the same platform fails open. How do you encode this criticality per model in the serving architecture?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Fail-open vs Fail-closed](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/Fail-open_vs_Fail-closed.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an ML feature and prediction pipeline that handles real-time fraud detection with fail-closed semantics while ensuring a recommendation system operates under fail-open conditions, we need to consider several key aspects of distributed systems engineering. The core challenge is balancing robustness in critical systems like fraud detection against the flexibility required for other services such as recommendations. This design will involve careful architecture choices, service isolation, and redundancy strategies.

First, let’s define what "fail-closed" and "fail-open" mean in this context:

Fail-closed: The system blocks transactions or actions when it detects a critical failure to ensure no potentially fraudulent activity is processed.
Fail-open: The system continues to operate as much as possible, even if some components are down.

#### Triage
Understanding the Question and Assumptions

The question probes:

Your understanding of fail-closed vs. fail-open principles.
How you would architect a distributed system considering different criticality levels for different services.
Techniques to manage service dependencies in such a complex environment.

Key assumptions include:

The fraud detection model is highly critical and cannot afford any risk of false negatives (i.e., allowing fraudulent transactions).
The recommendation system can tolerate some degradation without significant business impact.

#### First-Principles Solution
Service Isolation: Define clear boundaries between the fraud detection pipeline and the recommendation engine to prevent cascading failures.
Dependency Management: Implement strategies to ensure that the recommendation service remains operational even when critical dependencies are down.
Redundancy and Resilience: Design fault-tolerant mechanisms for both services to maintain high availability.
Service-Level Agreements (SLAs): Establish SLAs with clear definitions of what constitutes a failure condition, ensuring both systems have different criteria.
Detailed Architecture
Fraud Detection Pipeline (Fail-Closed)
Data Ingestion: Use robust streaming platforms like Apache Kafka to ingest real-time transaction data.
Feature Extraction: Implement feature extraction services that can operate independently but provide features crucial for fraud detection.
Model Serving: Deploy the ML model on a highly available and resilient platform, such as an ensemble of containers managed by Kubernetes.
Transaction Blocking: Integrate the prediction results directly into the transaction processing pipeline to block transactions immediately upon detecting potential fraud.
Recommendation Engine (Fail-Open)
Data Ingestion: Similar ingestion mechanisms can be used here, but with less stringent requirements for real-time data.
Feature Extraction: Use the same feature extraction services, though they might have fallbacks or alternate implementations to ensure availability.
Model Serving: Deploy recommendation models on a less critical service that can handle some downtime without impacting user experience.

#### Example
Architecture
Real-time Transaction Data
Kafka Topic
Feature Extraction Service 1
Feature Extraction Service 2
ML Model (Fraud Detection)
ML Model (Recommendation)
Transaction Processing Pipeline(Fail-Closed)
User Interface /Recommendation System(Fail-Open)

#### Common Mistakes
and Follow-Ups
Ignoring Service Dependencies: Failing to consider how one service’s failure can affect another.
Overcomplicating Fail-Open Systems: Making the recommendation system too robust, increasing complexity unnecessarily.
Lack of Redundancy Planning: Not adequately planning for potential failures in critical components.

#### Closing Summary

By carefully designing a distributed architecture that isolates and prioritizes services based on their criticality, we can ensure that the fraud detection model operates under fail-closed conditions while maintaining flexibility in the recommendation system. This approach requires thoughtful service design, robust feature extraction, and resilient deployment strategies to achieve the desired outcome.

**Comparison:**

```
Service	SLA Criteria	Failure Condition
Fraud Detection	High Accuracy and No False Negatives	Any feature extraction or model prediction failure
Recommendation	High Availability with Some Degradation	Feature extraction service failure
```

**Code:**

```python
# Example Feature Extraction Service
class FeatureExtractor:
    def __init__(self):
        self.features = {}

    def extract_features(self, transaction_data):
        # Extract features from transaction data
        pass

# Example Fraud Detection Model
class FraudDetectionModel:
    def predict(self, features):
        # Predict fraud probability based on extracted features
        return 0.9  # Placeholder value
```

**Example input / output:**

| Input (Transaction Data) | Output (Feature Extraction) |
|--------------------------|------------------------------|
| User ID: 12345, Amount: 150 | user_behavior: 0.8, transaction_amount: 150 |

**Data:**

```json
{
  "transaction_data": {
    "user_id": "12345",
    "amount": 150,
    "timestamp": "2023-10-01T10:00:00Z"
  },
  "features": [
    {"feature_name": "user_behavior", "value": 0.8},
    {"feature_name": "transaction_amount", "value": 150}
  ]
}
```

**Flow:**

1. Real-time Transaction
2. Kafka Topic
3. Feature Extraction Service 1
4. Feature Extraction Service 2
5. Fraud Detection Model
6. Transaction Processing(Fail-Closed)
7. Feature Extraction Service 3
8. Recommendation Model
9. User Interface /Recommendation System(Fail-Open)

