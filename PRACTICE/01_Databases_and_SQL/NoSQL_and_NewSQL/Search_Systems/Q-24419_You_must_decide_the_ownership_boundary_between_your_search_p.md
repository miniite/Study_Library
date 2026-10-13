---
type: "practice"
difficulty: "Expert"
---

# You must decide the ownership boundary between your search platform and the ML infra platform for feature stores that feed learning-to-rank. Where does data ownership, freshness SLA, and model-serving responsibility sit, and how do you avoid a fragile cross-team dependency?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Search Relevance](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/Search_Relevance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to define the boundaries of responsibility between our search platform and the ML infra platform regarding feature stores that feed learning-to-rank models. The key aspects are data ownership, freshness SLAs, and model-serving responsibilities. By clearly delineating these roles, we can ensure a robust system with minimal cross-team dependencies.

Firstly, let's clarify what needs to be done:

Data Ownership: Determine who is responsible for ingesting raw data into the feature stores.
Freshness SLA: Define how up-to-date the data must be and who ensures this.
Model-Serving Responsibility: Identify which team handles serving the model predictions.

To avoid a fragile cross-team dependency, we should design a system where each component is well-defined and self-contained as much as possible.

#### Triage
Reiterating the Question

The question revolves around establishing clear boundaries between two critical components of our architecture:

Search Platform: Responsible for providing real-time search capabilities.
ML Infra Platform: Manages feature stores, ML models, and model serving.

#### Assumptions

The feature stores contain historical data that is used to train learning-to-rank models.
The learning-to-rank models are periodically retrained using the latest available features from the feature stores.
Both platforms must ensure high availability and low-latency services.

#### Solution
From First Principles
1. Data Ingestion and Ownership

Search Platform:

Ingestion Role: Focuses on real-time indexing of data. It will not handle raw data ingestion from external sources or feature store updates.
Ownership: The ML Infra team is responsible for ingesting the raw data into the feature stores.

ML Infra Platform:

Ingestion Role: Manages the continuous integration and deployment (CI/CD) pipeline that pulls in new data, transforms it into features, and updates the feature stores.
Ownership: Owns the end-to-end process of ingesting raw data, transforming it into relevant features, and updating the feature stores.
2. Data Freshness SLA

Search Platform:

SLA Requirement: Ensures that the search results are as fresh as possible based on the latest available data in the feature stores.
Monitoring: Implements monitoring tools to alert when there is a delay or inconsistency between the time the ML Infra updates the feature stores and the time the search platform fetches it.

ML Infra Platform:

SLA Requirement: Ensures that feature stores are updated with fresh data within specified SLAs.
Monitoring: Implements internal monitoring to ensure timely updates of feature stores. This includes alerting mechanisms for any delays in ingestion processes.
3. Model-Serving Responsibility

Search Platform:

Serving Role: Uses the latest version of the learning-to-rank models served by the ML Infra platform.
Responsibility: Ensures that the search system is configured to fetch and use the correct model version from the ML Infra platform.

ML Infra Platform:

Serving Role: Manages serving the most recent learning-to-rank models.
Responsibility: Ensures that the models are served reliably, with low latency and high availability.

#### Closing Summary

By clearly defining these roles and responsibilities, we can create a robust architecture where:

The ML Infra platform handles data ingestion, feature store management, and model serving.
The search platform focuses on real-time indexing and ensuring the freshness of data used for queries.
Cross-team dependencies are minimized by well-defined interfaces and SLAs.

#### Common Mistakes/Follow-Ups
Failing to Define Clear Interfaces: Without clear APIs or protocols, teams might struggle with data synchronization issues.
Ignoring Data Freshness SLAs: Ensuring that both platforms meet their SLA requirements is crucial for system reliability.
Underestimating Model Version Management: Proper version control and rollback strategies must be in place to handle model updates without disrupting service.

To address these, we should implement robust monitoring and alerting systems across all components of the architecture.

**Comparison:**

```
Feature Store Data Schema	Description
id	Unique feature identifier
timestamp	Timestamp of the last update
features	Array of numerical features
Model Version History	Description
1.2	Latest version with updated model parameters
1.1, 1.0	Previous versions for rollback purposes
```

**Code:**

```python
# Example of a simple API for fetching feature store data (by ML Infra Platform)
def fetch_features(feature_id):
    # Implement logic to retrieve features from the database or cache
    pass

# Example of a search request handling function (by Search Platform)
def handle_search_request(query):
    # Fetch latest model version and use it to process query
    latest_model = get_latest_model_version()
    results = apply_model(latest_model, query)
    return results
```

**Example input / output:**

```

Input:

Search request with query q="best products".

Output:

Top-ranked search results based on the latest learning-to-rank model version.
JSON
Copy
{
  "results": [
    {"product_id": "12345", "rank_score": 0.98},
    {"product_id": "67890", "rank_score": 0.95}
  ]
}
```

**Data:**

```json
{
  "feature_store": {
    "data_schema": [
      {"id": "string", "timestamp": "datetime", "features": ["float"]}
    ],
    "freshness_sla": "60 seconds"
  },
  "model_versioning": {
    "latest_version": "1.2",
    "version_history": ["1.1", "1.0"]
  }
}
```

**Flow:**

1. Raw Data Ingestion
2. Feature Store Update
3. Feature Store Data
4. Learning-to-Rank ModelTraining
5. Model Serving
6. Search Platform
7. Real-time Indexing
8. Query Processing andResponse Generation
9. SLA Monitoring for Freshness
10. Versioned Model Fetch

